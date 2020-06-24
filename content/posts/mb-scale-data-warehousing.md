+++
draft = true
date = 2020-06-24T21:18:46+10:00
title = "Megabyte-Scale Data Warehousing"
description = "(Mis)Using petabyte-scale Cloud-based data warehouse tools  to make data central to your organisation."
tags = ["GCP", "BigQuery", "data"]
+++

Putting data at the heart of an organisation is key to success, especially so if you are a start up
or are rapidly evolving you products and business strategy. Readily available data helps you make
informed and logical choices quickly whilst removing some of the guess work.

However, making data accessible to everyone in your organisation can be a tough task. Often the data
resides in a complex mixture of internally developed systems, self-hosted products and third-party
Cloud apps. You never intended it to be this way but as the business grew and your needs changed you
kept adding one more tool. You've probably even found that different teams have introduced software
that only they use but contains some insights you'd love to have access to.

You might be thinking "this isn't me, I get a weekly report with all the data I need" but really
consider all the data in your business. Where are your sales orders? Customer data? Web page
analytics?  Do you have access to these? Is there data you know exists but you just can't get at?
To make things worse you're probably following security best practices and very
few people (if anyone) gets access to all systems.

# So how do you pull all this together?

You engineering team is probably busy working on that new feature and they are too busy to implement
that new change that adds a new column to your weekly report. god forbid you want a one off report
for a bespoke use case. Want to get that data into an operational dashboard, well thats even more work.

At reposit we've taken a different approach big query something something

Dont need to be a datawarehouse whizz at this scale. Just make the data accessible.



stitch/singer

custom scripts

datastudio
