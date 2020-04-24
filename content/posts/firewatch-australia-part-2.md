+++
draft = true
date = 2020-04-22T20:28:11+10:00
title = "Building Firewatch Australia, Part 2 - Scaling on the Cheap"
description = ""
slug = ""
tags = ["Firewatch Australia", "GCP", "serverless"]
externalLink = ""
+++

{{< seriesbanner >}}
This is part 2 in a series of 4 posts on building the [Firewatch Australia](https://firewatchaus.com/) app. Check out the
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
