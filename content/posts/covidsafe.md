+++
draft = false
date = 2020-04-26T16:00:00+10:00
title = "How COVIDSafe Works"
description = "How the Australian COVIDSafe app works and help keep users privacy to a maximum."
tags = ["Firewatch Australia", "GCP", "Cloudflare", "serverless"]
+++

{{< updatebanner >}}
**Update 27/04/20 14:30:** Originally this article was formed from analysis of the Singapore Government's
TraceTogether app from which COVIDSafe is based. I've since had time to better understand the differences
by reading the COVIDSafe [Privacy Impact Assessment](https://www.health.gov.au/resources/publications/covidsafe-application-privacy-impact-assessment) and have updated the details below as well as adding a section describing the differences.
{{< /updatebanner >}}

The Australian Government has just released the COVIDSafe app to help with contact tracing in
Australia during the COVID-19 pandemic. Here's a brief outline of how it works:

- You download the app from the Play Store or App Store.
- You then register in the app with a phone number, name, age range and postcode. Your phone number
  is transmitted to the a secure server and recorded with a unique User ID.
- The app on your phone gets sent a Unique ID valid for two hours from the server. When receiving
  the Unique ID the app will respond to the server effectively stating "Yes, I received a new ID".
- Every two hours the app will request a new Unique ID from the server.
- If you do not have an internet connection or the app is not running the previous Unique ID will
  continue to be used.
- The "Yes, I received a new ID" reports will be used by the Department of Health to determine how
  many people are using the app and how many people have it actively enabled.
- When two people using the app come into contact the Unique IDs, Bluetooth signal strength and
  date/time of the encounter are recorded on both devices. This happens every one minute whilst the
  two people are in contact.
- Each phone keeps a copy of all the encounters. This log of encounters is kept only on the phone
  and is not sent to the server without permission.
- After 21 days each encounter record will be automatically deleted.
- If you test positive for COVID-19 you can choose to transfer the history of encounters to the server.
- The Health Authority can match the encounters you have uploaded, to the User IDs and phone numbers
  used to register and start contacting the people you have been in contact with that may have been
  exposed to COVID-19.

Here are some other interesting points of the implementation:

- The implementation itself has a fairly high degree of privacy but you do have to have a fair
  amount of trust and confidence that it all works as advertised and that the data is being used for
  this one purpose only.
- On Android the requested permissions include location. This is related to Bluetooth permissions
  and no location data is actually obtained.
- COVIDSafe considers "close contact" as exposure of 1.5m for a period of 15 minutes or more. The
  distance between people is measured using Bluetooth signal strength. Seeing as the Bluetooth signal
  strength is recorded as part of the encounter and uploaded to the server if you test positive, it
  looks like the calculation of signal strength-to-distance is something that happens after
  you test positive. The same appears true of the 15 minute encounter duration. Recommendation 18 of
  the [COVIDSafe PIA](https://www.health.gov.au/resources/publications/covidsafe-application-privacy-impact-assessment)
  would seem to confirm this.
- Given that this requires an app to be in installed, it would seem that phone numbers shouldn't be
  mandatory and the whole process could be kept anonymous by using push notifications. At least
  making this an option would be an improvement.
- Name absolutely does not seem required for this to work - so feel free to use a fake one. I'd love
  to see the database of hilarious fake names at the end of this.
- Daily notifications will appear on your phone so you can have confidence that the app is running
  in the background correctly.

Differences between COVIDSafe and TraceTogether:

- COVIDSafe uses 2 hour Unique IDs rather than the 15 minutes used by TraceTogether. This may seem
  trivial but the privacy implications are fairly big. This makes the window of time that third party
  tracking devices could detect and correlate you much larger meaning it would be easier to track
  your location from place-to-place. I'd make a guess that in 2 hours you could visit a few different
  places where you could be tracked, but in 15 minutes you're much less likely to.
- TraceTogether issues Unique IDs in batches and gives you enough for the next 24 hours. COVIDSafe
  issues one at a time. With TraceTogether, this means if you do not have an internet connection
  you should  have plenty of Unique IDs to operate as normal for up to the next day. Whereas with
  COVIDSafe it is much more likely to have to use a Unique ID for longer than the 2 hours if no internet
  connection is available. Also, this allows the servers of COVIDSafe to track your app usage in
  more detail, they expect to be contacted every 2 hours rather than every 24. This may be an
  advantage to the goal of the project as they have more data on uptake an usage but a disadvantage
  on privacy. This latter point seems fairly trivial.
