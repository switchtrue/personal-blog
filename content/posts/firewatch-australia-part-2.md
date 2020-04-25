+++
draft = false
date = 2020-04-25T01:00:00+10:00
title = "Building Firewatch Australia, Part 2 - Scaling on the Cheap"
description = "Using Cloudflare caching to dramatically reduce the number of requests to paid-for infrastructure."
tags = ["Firewatch Australia", "GCP", "Cloudflare", "serverless"]
+++

{{< seriesbanner >}}
[Firewatch Australia](https://firewatchaus.com/) is a free app released over the 2019-2020 Australian
Bushfire season to help track bushfires. This is part 2 in a series of 5 posts on building the app.
Check out the [introduction]({{< ref "/posts/firewatch-australia-introduction.md" >}}) or take a
look at the [previous post on Data Processing]({{< ref "/posts/firewatch-australia-part-1.md" >}})
to learn more.
{{< /seriesbanner >}}

---

I wasn't expecting many users when I launched the app, maybe just a few friends, but nonetheless I
wanted to make sure it scaled well and didn't break the bank in terms of infrastructure costs.
Turns out this was a good play as the app had several thousand users and tens of thousands
of requests just a few days after release.

In the [previous post]({{< ref "/posts/firewatch-australia-part-1.md" >}}) I documented the process
for gathering the data using [Google Cloud's][0] serverless [Cloud Functions][1] so it might not surprise you to know
that I wanted to serve the data via a similar method. I used functions with HTTP triggers
to return JSON which served as the API for the app. The functions themselves were pretty simple -
they queried data from [Firestore][2] and returned it.

# Caching

The data in Firewatch Australia is fairly static. Any given fire only tends to be updated every
few hours or so, it depends on how quickly the fire is changing and all users are served the same
data so they see the same list of fires. As such it's a great candidate for caching. The goal here
is to serve as many requests as possible from a cache so that it's not constantly hitting the
functions and the database incurring both time and cost.

[Cloudflare][3] has a great caching service that is outrageously simple. There's really very little
you have to do to set it up:

- You set up Cloudflare as the name servers for your domain.
- You configure a CNAME record in Cloudflare's DNS that contains a URL that traffic should be
  proxied to.
- Requests now go to Cloudflare servers which act as a proxy.
- If the request isn't in the cache, Cloudflare will make a request to your API and cache the
  response.
- If the request is in the cache Cloudflare returns it without ever hitting your API

This works really well. The graph below is straight from the Cloudflare console and shows that the
majority of requests are hitting the cache - roughly 92%.

{{< image
      class="center"
      src="/images/firewatch-cloudflare-console-1-day.png"
      caption="Screenshot of the Cloudflare UI for a single day showing ~92% cache hit rate with a peak of 3100 requests per hour"
      alt="Screenshot of the Cloudflare UI for cached vs uncached requests" >}}

Cloudflare have [edge nodes located all over the world][4] and, importantly for this app, across
Australia. This means Cloudflare is also able to serve the content quickly by serving it from the
location nearest the user. This network coupled with the fact we caching the data results in a very
significant performance increase. In the screen shot below you can see its 2-3 time faster when
hitting the Cloudflare-backed customer domain vs hitting the Cloud Function directly.

{{< image
      class="center"
      src="/images/firewatch-cloudfare-speed-test.jpg"
      caption="Speed test of curling the Cloud Function vs the cached Cloudflare content. It saves about 3.6 seconds."
      alt="Commandline speed test showing an elapsed time of 5.234 second for the raw function and 1.619 second for the Cloudflare cache" >}}

Remember, all I've had to do here is sign up to Cloudflare and configure my nameservers. There's
absolutely no code required up to this point. It's really quite impressive. If that wasn't enough,
it's also totally free. Cloudflare has one of the most impressive free tiers of any product I've
used - up to the point where I was sceptical - but their [CEO outlines some really good reasons for
this][5].

### Infinite Scale?

I know better than to suggest anything might actually scale infinitely, but certainly for my purposes
the API is at a point where scale is unlikely to ever be an issue. The more users the app has the more
cache hits there will be so it's really at the mercy of Cloudflare's scale which I'm sure is massive
and [easily handles far more traffic][6] than my app.

If requests do make it to Google Cloud then it's on a fully serverless stack. I would imagine the
bottleneck would be Firestore but interestingly the documentation for Firestore doesn't mention
limits on reads, but the [writes limits are 10,000 per second][7] which I don't think I'd ever
exceed.

This isn't a silver bullet either, the data in Firewatch Australia just happens to lend itself very
nicely to this approach.

# DNS with Cloud Functions

Cloud Functions with an HTTP trigger have a URL in the form:

    https://<compute-region>-<gcp-project-name>.cloudfunctions.net/<function-name>

Because this URL is unique and contains the details to locate the function its kind of important.
As such, Cloud Functions don't support a custom CNAME resolving to them, they fail with a 404.

Luckily, Google have a solution for this with Firebase functions. This is a little bit of extra work
that I wish I didn't have to do but it's pretty quick and easy.

1. Download the [Firebase CLI][9]
2. Login to the CLI with `firebase login`
3. Use `firebase init` to initialise a Firebase project
4. Edit `firebase.json` to configure your functions. See the example below.
5. Connect your [custom domain to Firebase][10]

Example `firebase.json` configuration:
{{< highlight json >}}
{
  "hosting": {
    "public": "public",
    "ignore": [],
    "rewrites": [
      {
        "source": "incidents",
        "function": "incidents"
      }
    ]
  }
}
{{< / highlight >}}



A (fairly big) downside to this is that I appear to be paying egress charges twice - once from the
Cloud Functions to Firebase and again from Firebase to the internet (or Cloudflare in this case).
I'm not sure if this is something specifically related to how I've set it up but I will
investigate this in the future.

{{< image
      class="center"
      src="/images/firewatch-billing-double-egress.png"
      caption="A Google Cloud billing report showing two lots of egress charges with suspiciously similar usage amounts from the two services traffic flows through."
      alt="Commandline speed test showing an elapsed time of 5.234 second for the raw function and 1.619 second for the Cloudflare cache" >}}

# Cache Invalidation

> There are only two hard things in Computer Science: cache invalidation and naming things.
>
> -- Phil Karlton

I don't remember how I came up with the name "Firewatch Australia" but I don't think I spent a great
deal of time on it. Fortunately, cache invalidation for this app is also pretty easy. Theres' a pretty
clear trigger - when a fire is updated and stored in the database (see the previous post for more on
this).

Cloudflare provides a simple API for invalidating certain URLs. This is nice because each time a fire
changes I just need tio invalidate the endpoints for that fire and the full list of fires. All other
fires can remain in the cache. The request is as follows - you need to supply the zone-id as well as
some authentication headers.

{{< highlight http >}}
POST https://api.cloudflare.com/client/v4/zones/:zone-id/purge_cache

{
    "files": [
        "https://firewatchaus.com/incidents",
        "https://firewatchaus.com/incidents/1233456,
    ],
}
{{< /highlight >}}

# Wrapping Up

The data in Firewatch Australia lent itself really nicely to a straight forward caching solution.
Using Cloudflare for this was incredibly simple and easily achieved my original goal of trying to
avoid as much traffic as possible hitting my Google Cloud infrastructure.

I'm pretty happy to say that all this work to handle scale is completely unused right now. The number
of users of the app has dropped to the low tens due to the excellent work our [Firies][8] put in
bringing the bushfires under control - theres simply nothing in the app to see!

In the next post I'll be talking about how the app itself was made using React Native and Expo.


[0]: https://cloud.google.com/
[1]: https://cloud.google.com/functions
[2]: https://cloud.google.com/firestore
[3]: https://www.cloudflare.com/
[4]: https://www.cloudflare.com/network/
[5]: https://webmasters.stackexchange.com/a/88685
[6]: https://www.troyhunt.com/serverless-to-the-max-doing-big-things-for-small-dollars-with-cloudflare-workers-and-azure-functions/
[7]: https://cloud.google.com/firestore/quotas#writes_and_transactions
[8]: https://www.urbandictionary.com/define.php?term=firies
[9]: https://firebase.google.com/docs/cli
[10]: https://firebase.google.com/docs/hosting/custom-domain
