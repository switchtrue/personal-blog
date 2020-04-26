+++
draft = false
date = 2020-04-26T16:00:00+10:00
title = "How COVIDSafe Works"
description = "How the Australian COVIDSafe app works and help keep users privacy to a maximum."
tags = ["Firewatch Australia", "GCP", "Cloudflare", "serverless"]
+++

The Australian Government has just released the COVIDSafe app to help with contact tracing in
Australia during the COVID-19 pandemic. Here's a brief outline of how it works:

- Firstly, you download the app from the Play Store or App Store.
- You then register in the app with a phone number, name and postcode. Your phone number is transmitted to the a secure server and recorded with a unique User ID.
- The app on your phone gets sent a batch of encrypted temporary IDs. These temporary IDs are used to identify
  your app to other users and are only valid for 15 minutes. The temporary IDs are a derivative of
  your User ID which means the Health Authority is able to convert them back to your User ID and thus
  name and phone number.
- If you run out of temporary IDs you get more from the server.
- Every time you come into contact with someone else using the app your current temporary IDs (the
  ones valid for the current 15 minute period) are exchanged.
- Each phone keeps a copy of all the temporary IDs it has ever encountered. This log of temporary
  IDs is kept only on the phone and is not sent to the server without permission.
- If you test positive for COVID-19 you can choose to transfer the history of temporary IDs you've
  encountered to the server (including the time of the encounter).
- The Health Authority can now decrypt all the temporary IDs you have uploaded, convert them back to
  the phone numbers used to register and start contacting the people you have been in contact with
  that may have been exposed to COVID-19

Here are some other interesting points of the implementation:

- The implementation itself has a fairly high degree of privacy but you do have to have a fair
  amount of trust and confidence that it all works as advertised and that the data is being used for
  this one purpose only.
- You get a batch of temporary IDs from the server so that even if you have a poor or no network
  connection the app can still work by changing your temporary ID every 15 minutes and maintain
  privacy.
- On Android the requested permissions include location. It seems that this is related to Bluetooth
  permissions and no location data is actually obtained.
- COVIDSafe considers "close contact" as exposure of 1.5m for a period of 15 minutes or more. The
  distance between people is measured using Bluetooth signal strength. It's not clear if the 15
  minute period is something that is determined on-device, and therefore temporary IDs are only
  stored after 15 minutes or if it's something that is determined later and temporary IDs are
  recorded immediately.
- Given that this requires an app to be in installed, it would seem that phone numbers shouldn't be
  mandatory and the whole process could be kept anonymous by using push notifications.
- Name and postcode absolutely do not seem required for this to work - so feel free to use fake
  ones. I'd love to see the database of hilarious fake names at the end of this.
- Daily notifications will appear on your phone so you can have confidence that the app is running
  in the background correctly.

*Note: COVIDSafe is based on the Singapore Government's TraceTogether app and open BlueTrace
algorithm. This article assumes that COVIDSafe has not altered the implementation and until the
source code becomes available it will be difficult to tell. That said, some folks have already
managed to take a look and the implementation seems to be the same.*
