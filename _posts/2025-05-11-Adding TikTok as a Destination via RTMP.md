---
title: "Adding TikTok as a Destination via RTMP"
date: 2025-05-11 08:42:00 -500
categories: [StreamYard, A/B Testing, Destinations]
tags: [production, new core feature, my journey, SQL]
image: /assets/img/StreamYard/TikTok/tiktok_rtmp.png
---

This has been my favorite task at Bending Spoons yet! As you may know, the core functionality of StreamYard is allowing users to stream to different destinations simultaneously. As such, getting the chance to add a destination like this allowed me to get a brilliant overview and insight of many services in the codebase, and most core functionalities. It was both a fun task and great learning opportunity, and I'm super grateful to my colleagues for letting me steal this one from backlog hehe.


## How Forwarding Streams Works: An Overview

In a nutshell, what we do is allow the user to enter a studio to set up a stream or recording, and then forward this broadcast to the given destination. There are 2 main ways in which we can do this: the first is via an API, in which case we usually need to have access to the platform's docu and possibly be in contact with one of their dev teams to implement it and set up a testing account. For more details on this, the next article will go over how I implemented one of the APIs for Kick, specifically for category management. The second is via RTMP, which stands for Real-Time Messaging Protocol.

This streaming protocol was originally developed by Adobe. It allows us to transmit audio, video, and data over the internet in real-time. It works by establishing a persistent connection between a streaming source and a server - essentially, we take your video input, encode it, and push it through RTMP streams to multiple destinations simultaneously. 
This allows for multiplatform, low-latency live content.

To set this up, you generally need 2 things: a server URL (usually starts with `rtmp://` or `rtmps://`), and a stream key, which are provided by the platform in question, in this case TikTok. So ideally, it is possible to add infrastructure for this kind of functionality that is platform-agnostic. And StreamYard has always had this. There is a (paid) "Custom" destination available which already allows users to stream via RTMP, but the UI is generic and it is passed in the backend as `platform: custom`, meaning we don't have as much information about the different destinations used, nor do we offer a personalized experience. This was my goal; not only adding TikTok as a destination (my task), but managing to add a framework that would differentiate the destinations as much as possible, while being generalizable enough to allow us to add other destinations down the line with minimal changes. 

![Desktop View](/assets/img/StreamYard/TikTok/tiktok_announcement_modal.png){: .normal}


## Backend Implementation

The first thing I did was find which functions exactly we used to create and update outputs, and create services to call these with added values to the payload, the server URL and stream key. This includes things like building the stream URL, making these streams default to portrait orientation in the studio as this is TikTok's main usage, and leveraging most of the existing RTMP services for the new interface.

Then most of the work was validations: making sure `'tiktokRtmp'` was an accepted platform in all necessary schemas, and adding a regex for the server URL and stream key. I made sure to add a requisite to the server URL to have to include the string `.tiktok`, as "Custom" destinations are only for subscribed users, and I needed to avoid people abusing the new TikTok implementation to just stream to all destinations for free, as long as they put up with a TikTok-specific UI on StreamYard.

I called the new interface `FakeViaRTMP`, as we use it for destinations we want to show as "separate" destination but in reality just use RTMP behind the scenes. Say we wanted to add another of these,say Telegram, we would now just need to declare Telegram to use the `FakeViaRTMP` interface, then add `'telegram'` in all the validations, and we would be done!


## Frontend Implementation

For the frontend, we needed a new flow to add the server URL in a different step of the usual modal used to create broadcasts. In comparison, we have below the flow for the Custom destinations:

![Desktop View](/assets/img/StreamYard/TikTok/custom_add_destination.png){: .normal}

Where we add the server URL and stream key first to create the output, and then:

![Desktop View](/assets/img/StreamYard/TikTok/custom_added.png){: .normal}

We can just go live, versus the flow for TikTok via RTMP:

![Desktop View](/assets/img/StreamYard/TikTok/tiktok_add_destination.png){: .normal}

Where we firstly just need to add a "nickname", which is just for StreamYard to show, not on the actual platform, and then:

![Desktop View](/assets/img/StreamYard/TikTok/tiktok_added.png){: .normal}

We then see the output created and the instructions on how to find our server URL and stream key, which we can then add. Again, I added switch cases for the instruction strings and any other error messages or TikTok specific labels to make it easily scalable and reproducible. I had to add state management too here to ensure it populates when we edit the destination, so users don't have to re-input their details each time.

Finally, I made sure to add the necessary warnings or info tooltips that other platforms have:

![Desktop View](/assets/img/StreamYard/TikTok/tiktok_other.png){: .normal}

Here you can see how the destination looks once added, that it indeed defaults to portrait upon entering a TikTok studio, and the informative table of which destinations allow connected comments (that is, if a user posts a comment on the actual platform, do you see it in the StreamYard studio in real-time?).

Finally, I added metrics to see whether this new destination was actually actively used by users, and whether separating it from "Custom" had an impact on usage. Here we only compared paying users, as obviously users on a free plan can't access "Custom" to begin with. Spoiler alert: it does matter.

To continue adding our hypothetical new destination of Telegram, it would now just be a matter of adding the platform icon, adding all the necessary strings to instructions, warnings and so on, as well as metrics, and adding `'telegram'` to the array of destinations shown on the modal. 


## Testing

One of the biggest challenges of this task was that it was quite critical - any changes to the backend `createOutput` or `updateOutput` could have potentially affected all users and broken the core feature of StreamYard, streaming - but it was not easily testable.

In theory, this shouldn't have been hard to test - just set up a stream and see what happens. And for the most part, setting up a fake stream with made up credentials works to test most of the UI. But testing whether or not the actual forwarding of the stream to TikTok's servers worked was a different issue. For this, we needed a TikTok testing account, and considering they wouldn't provide one and that you need 1000 followers to even go live on TikTok, this was not easy to come by. I did enlist one of my "influencer" friends for this, kidnapping her account with access to TikTok Live for testing, but apparently you also need a Windows and TikTok Studio, and to follow a number of steps until you can access your key. We were not eligible in the end. 

So I decided, how bad can it be if I just release? And as the memories of Vietnam flooded my brain with past times StreamYard has had any issue, I decided against it. 

To be fair, I had a fair idea that it was going to work - in dev, I had turned off the TikTok specific validation requirements and managed to connect my YouTube account using the TikTok interface, so it would most probably work with a different server URL.

![Desktop View](/assets/img/StreamYard/TikTok/tiktok_yt_testing.png){: .normal}

However, I wanted to be cautious, so I added the new destination under a FF and fled to our database on GCP. I wanted to find some users that had recently streamed to TikTok through "Custom", and segment them first as an initial group of beta testers. Since we had recently ran a migration, I could just use BigQuery to find 200 users that had streamed to TikTok in the past 3 months by filtering the stream URLs containing "tiktok". I then closely monitored incoming errors for any platform-specific alerts we received. This was the most foolproof plan I could come up with given that actual testing was off the table.

Edit: 2 months later, no bugs reported, and users are happy!


## Conclusion

For any devs out there who know the pain of implementing something without testing (nothing ever works on the first try), I am very happy that it is ~~initially~~ working well, as users have not yet reported any issues. Especially considering the scale and fragility of this task, I am very proud of the outcome! And can't wait to add some more Fake Via RTMP destinations - the framework should make future integrations much smoother!

