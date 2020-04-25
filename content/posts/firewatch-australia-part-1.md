+++
draft = true
date = 2020-04-25T00:00:00+10:00
title = "Building Firewatch Australia, Part 1 - Data Processing"
description = "How the data capture and processing pipelines work in a serverless Google Cloud setup for the Firewatch Australia app."
tags = ["Firewatch Australia", "GCP", "serverless"]
+++

{{< seriesbanner >}}
[Firewatch Australia](https://firewatchaus.com/) is a free app released over the 2019/20 Australian
Bushfire season to help track bushfires. This is part 1 in a series of 5 posts on building the app.
Check out the [introduction]({{< ref "/posts/firewatch-australia-introduction.md" >}}) or take a
look at the [next post about scaling on the cheap]({{< ref "/posts/firewatch-australia-part-2.md" >}})
to learn more.
{{< /seriesbanner >}}

---

When I first started building Firewatch Australia I was very much building the plane whilst flying it.
I had a few goals in mind but didn't know exactly how they would manifest or how I would implement
them. One early feature I knew I wanted was a visual history of each fire showing its progression.
This seemed important to me so that you could better understand the speed at which a fire is
spreading or if it has slowed down.

The RFS feed only offers the current size and shape of the fires so in order to achieve this I
needed to start scraping and saving the data before deciding how I was going to manage and serve
it long term. I also needed to start gathering the data ASAP so that history would be
available when the app launched.

{{< video
      class="center"
      src="/videos/firewatch-australia-timelapse.mp4"
      caption="Video of the app showing a timelapse of the Clyde Mountain Fire progression over 5 days." >}}

# Gathering Data

I wanted to go serverless for the whole architecture for various reasons but mostly time to market
(even though it's free!) - I needed to get this app out ASAP for it to be useful to people.
No servers to setup is a real time saver and it means you can spend all your time writing code
and adding features. I also wanted to use [GCP][1] as thats where I'm most comfortable.

Google Clouds serverless compute product is simply called [Cloud Functions][2] and pretty much does
what it says on the tin - you deploy individual functions to run and they can be triggered through
a variety of sources including HTTP endpoints and events within GCP. No servers to mangage and you
pay per 100ms of execution with a generous perpetual [free teir][3].

I created a simple Cloud Function to make a request to the RFS feed and fetch the data. The response
is [GeoJSON][4] and contains a list of all the currently active fires. I wanted to make sure that
each fire was captured seperately and only make a new record when it changed - I didn't want storage
costs to blow out by storing the same data everytime it was fetched.

To do this the Cloud Function splits the data up into each fire and generates an md5 hash of the
GeoJSON for that fire. The GeoJSON for each individual fire and its md5 hash we then written directly
to [Cloud Storage][5] using the [python client][6]. The next time the GeoJSON is fetched the hash
of the latest data is compared with that in storage to determine if the fire has changed, if it
has we write the new data and update the hash.

{{< image
      class="center"
      src="/images/firewatch-incidents-bucket.png"
      caption="Listing of a Cloud Storage bucket for a single fire showing one file for each change in its history and the md5 hash."
      alt="Image the contents of a Cloud Storage bucket showing a fires history of changes and the md5 hash file" >}}

Next I needed a way to trigger this periodically. Google have a product called [Cloud Scheduler][7]
which they describe as a "Fully managed cron job service". This is a really great service that often
seems overlooked and little talked about. It allows you to specify a crontab syntax for when job
should run and provide something to trigger such as an HTTP endpoint which is perfect for Cloud
Functions. As an added bonus you can even specify a timezone along with your crontab so, if you're
lazy like me, you don't have to do the timezone conversions yourself.

It doesn't hurt that you get [3 free Scheduler jobs][8] and only 10c per job after that - thats 10c
per job, not per exectution, so if you run it once a month or thousands of times it still only costs
10c. Note that you do pay for any Cloud Function resources you consume if a Cloud Function is your
target.

I wanted to refresh the data every 5 minutes so that it was never too far out of date. A simple
crontab of `*/5 * * * *` does this and I made the target the HTTPS endpoint of the Cloud Function.

{{< image
      class="center"
      src="/images/firewatch-cloud-scheduler.png"
      caption="Screenshot of the configuration of the Google Cloud Scheduler job. Here you can see the crontab (with timezone 👍) and Cloud Function being used as a trigger."
      alt="Screenshot of the configuration of the Google Cloud Scheduler job" >}}

At this point I've deployed a single Cloud Function, scheduled it periodically and am starting to
gather history into Cloud Storage. This was really quick in a serverless world on GCP and I quite
literally did it in a couple of hours whilst enjoying a few of Christmas beers!

Having every distinct change for each fire being written to Cloud Storage periodically forms a
really solid base for what comes next.

# Processing Data

Cloud Function support more than just HTTP triggers, another super useful trigger is "Cloud Storage".
This allows you to trigger a function based on an object event in a Cloud Storage Bucket - you can
trigger a function when an object is created, deleted or archived.

{{< image
      class="center"
      src="/images/firewatch-cloud-storage-trigger.png"
      caption="Screenshot of a Cloud Function trigger configuration that fires when an object is created in a Cloud Storage Bucket."
      alt="Screenshot a configuration for a Cloud Storage trigger for a Cloud Function" >}}

Using this trigger type coupled with having each distinct change in Cloud Storage means I can process
each change as it arrives. A Cloud Function receives the metadata about the newly created file (i.e.
a brand new fire or a change to an existing fire), fetches that file from Cloud Storage procesess it
into a format suitable for the app and stores it to [Cloud Firestore](https://cloud.google.com/firestore).

Firestore is Googles serverless NoSQL document store product. It organises data in "documents" which
is a set of fields with values and "collections" which is a group or list of documents. One nice
feature is that collections can be nested in documents as "subcollections".

Processing the data from Cloud Storage and writing it to Firestore boils down the to the following steps:

1) Check if the fire already exists in Firestore, if it doesn't then insert a new document in a
   root-level collection called "incidents". Populate this document with some basic information.
2) Each fire document has a "history" subcollection to contain all the changes over time. Insert
   the lastest data into this subcollection indexed by the timestamp that the update occurred at.

Using subcollections here has a couple of great advantages here. Firstly, the parent document can be
kept small so when loading the main screen of the app we don't have to fetch the full history for
every fire. Secondly, subcollections can be inserted into without having to fetch the parent document
or the rest of the history subcollection. Some of these fires lasted for more than 2 months with
regular updates of GeoJSON meaning that the documents were pretty big, not having to fetch them all
the time was a big help.

{{< image
      class="center"
      src="/images/firewatch-firestore-ui.png"
      caption="Screenshot the Firestore UI showing some root-level collections on the left, a list of incidents/fires in the middle, and fields contained in an incident on the right. The right panel also shows the history subcollection at the top."
      alt="Screenshot the Firestore UI" >}}

## Why not go Straight to Firestore?

This is a good question - why did I write to Cloud Storage first only to immediately read it and
write it to FireStore? Couldn't I have gone direct to Firestore?

That would have been an option, but as mentioned at the beginning I was very much figuring out what
I was doing as I doing it. I didn't know the final schema for Firestore when I started. I was also
not familiar with the RFS data feed nor with GeoJSON which meant if I did too much processing upfront
I was nervous that something would change and it would all break. Storing the raw data first meant I
could change my mind about processing and the Firestore schema and simply run the functions again
on top of the raw data. This turned out to be a very robust solution.

# Wrapping Up

I really liked the simplicity of this solution - there are two functions with two distinct roles -
one fetches data, the other processes it into something more useful. The triggers on Cloud Functions
as well as Cloud Scheduler make this super simple and the whole thing runs reliably in severless
environment.

In the next post I'll be talking about how this data was served via APIs and how I managed scale on
the cheap.



[1]: http://cloud.google.com/
[2]: https://cloud.google.com/functions
[3]: https://cloud.google.com/functions/pricing#free_tier
[4]: https://geojson.org/
[5]: https://cloud.google.com/storage
[6]: https://googleapis.dev/python/storage/latest/client.html
[7]: https://cloud.google.com/scheduler
[8]: https://cloud.google.com/scheduler/pricing
