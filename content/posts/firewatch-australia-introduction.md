+++
draft = false
date = 2020-04-22T20:23:30+10:00
title = "Building Firewatch Australia - Introduction"
description = ""
slug = ""
tags = []
categories = []
externalLink = ""
series = []
+++

{{< seriesbanner >}}
This is the introduction of a series of 4 posts on building the [Firewatch Australia](https://firewatchaus.com/) app. Take a look at the [next post on the data processing pipelines]({{< ref "/posts/firewatch-australia-part-1.md" >}}) to learn more.
{{< /seriesbanner >}}

---

In December 2019 my parents travelled accross the world from the UK to Australia paying me a visit for the first time in 8 years.
We had an intense few weeks planned travelling the east coast from Cairns to Gold Coast to Sydney to Narooma. During this time the [bushfires hit hard](https://en.wikipedia.org/wiki/2019%E2%80%9320_Australian_bushfire_season) taking many lives and properties in the [worst bushfire season Australia has ever seen](https://www.abc.net.au/news/science/2020-03-05/bushfire-crisis-five-big-numbers/12007716). We were unsure whether the planned trip could be completed safely and I spent a lot of time switching between different apps and websites trying to obtain data on active fires and road closures.

Whilst flicking between these various apps I realised that most of the data was open and I could pull
them together into a single source to make my life easier. If it was useful to me it might very well
be useful to others and I could do some good to help people during this time in the form of a free app.

Over the Christmas period I built the [Firewatch Australia](https://firewatchaus.com/) app bringing together full history of fires
from [RFS](https://www.rfs.nsw.gov.au/), [NASA](https://earthdata.nasa.gov/earth-observation-data/near-real-time/firms) locations of the active fire fronts and live details of current road closures
from [Live Traffic NSW](https://www.livetraffic.com/desktop.html).

Thanks to the hard work of my partner and her family the app had over 3000 active users in just a
couple of days with positive feedback that it was helping. It spent a brief amount of time in
the Top Charts on the App Store for reference apps even beating out the Bible!

{{< image
      class="center"
      src="/images/firewatch-app-store-top-charts.png"
      alt="Image of app store top chart listing showing Firewatch Australia in the number 3 spot." >}}

This series of posts covers the tech stack used to build Firewatch Australia including:

- [Part 1 Data Processing]({{< ref "/posts/firewatch-australia-part-1.md" >}}) - the serverless backend data processing pipelines on [GCP](cloud.google.com)
- [Part 2 Scaling on the Cheap]({{< ref "/posts/firewatch-australia-part-2.md" >}}) - serving the data efficiently and cost effectively with GCP and [Cloudflare](https://www.cloudflare.com/)
- [Part 3 The App]({{< ref "/posts/firewatch-australia-part-3.md" >}}) - building the app itself with [Expo](http://expo.io/)
