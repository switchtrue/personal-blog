+++
draft = true
date = 2020-04-25T06:00:00+10:00
title = "Building Firewatch Australia, Part 5 - The Data"
description = ""
tags = ["Firewatch Australia", "GCP", "serverless"]
+++

{{< seriesbanner >}}
[Firewatch Australia](https://firewatchaus.com/) is a free app released over the 2019-2020 Australian Bushfire season to help track bushfires. This is the final part in a series of 5 posts on building the app. Check out the [introduction]({{< ref "/posts/firewatch-australia-introduction.md" >}}) or take a look at the [first post on data
pipelines]({{< ref "/posts/firewatch-australia-part-1.md" >}}) to learn more.
{{< /seriesbanner >}}

---


RFS
 - has it problems, shapes disappear, bad
 - fires randomly start again with a new ID - handle manually and merge
 - ACT AES dont like it because AES is more up to date but they done have a feed of their own


NASA FIRMS
 - Good but pretty sure I used the wrong one, the other one refreshes faster
 - The aussie website is better because it already aggregates


Live traffic NSW
 - easy peasy, no issues at all
