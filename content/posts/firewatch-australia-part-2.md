+++
draft = true
date = 2020-04-25T01:00:00+10:00
title = "Building Firewatch Australia, Part 2 - Scaling on the Cheap"
description = ""
slug = ""
tags = ["Firewatch Australia", "GCP", "serverless"]
externalLink = ""
+++

{{< seriesbanner >}}
[Firewatch Australia](https://firewatchaus.com/) is a free app released over the 2019-2020 Australian Bushfire season to help track bushfires. This is part 2 in a series of 5 posts on building the app. Check out the
[introduction]({{< ref "/posts/firewatch-australia-introduction.md" >}}) or take a look at the [next
post on building the app]({{< ref "/posts/firewatch-australia-part-3.md" >}}) to learn more.
{{< /seriesbanner >}}

---

- background - wanted to prepare for scale just in case
- wanted to not pay a fortune in cloud bills

- cloudflare set up
- cloud functions custom DNS doesnt work with cloudflare
- used firebase functions
- paying for egress twice
- cloudflare graphs
- invalidating the cache
