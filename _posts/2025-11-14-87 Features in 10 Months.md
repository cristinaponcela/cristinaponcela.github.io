---
title: "87 Features in 10 Months"
date: 2025-11-14 08:42:00 -500
categories: [StreamYard, SEO, UA]
tags: [production, localized pages, backlings, hreflangs, canonical tags, indexing pages, my journey]
image: "https://raw.githubusercontent.com/cristinaponcela/cristinaponcela.github.io/refs/heads/main/assets/img/StreamYard/"
---

After a year full of learning and coding adventures, my time at Bending Spoons has come to an end as I pursue a different venture. Thanks to its extremely dynamic and autonomous culture, in the last 10 months (been here for 12, but the first 2 were training), I have built and shipped 87 features completely independently. This has amounted to a total of, drum roll, 214 PRs!

This article is thus a bit of a feature dump, more for myself than anything, with the highlights of things I've built and feel proud of. I obviously won't include any features already mentioned in other articles, and bear in mind that not all features are extremely impactful, or even user facing. But I can proudly go having built at least 25 very impactful and well-received features under my built, which alone would be impressive to me in just 10 months :D


## The Right Aside Side Tab Animation

I am a big fan of relatively small changes that modernize the product and make it aesthetically pleasing. This small feature scratches my brain each time I see it:

<iframe class="embed-video" loading="lazy" src="/assets/img/StreamYard/FeatureDump/side-bar-animation.mp4" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen=""></iframe>


## Blur Intensity Slider

Before, StreamYard only offered one setting for blurring your background, and it was waaaay too intense, it looked horrible. I realized this while using the product on a weekend while helping some friends out with something, and had a lightbulb moment where I realized "hey, I'm a StreamYard dev, if I don't like it I can change it". So I added a slider to adjust the intensity of the blur.

It was not trivial since this meant changes to the UI and backend as I needed to ensure the background changes propagate to the video that is recorded. But arriving the next week with a new feature, and the happiness of my colleagues, made it super worth it.

<iframe class="embed-video" loading="lazy" src="/assets/img/StreamYard/FeatureDump/blur-intensity-demo.mp4" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen=""></iframe>

And the polished UI:

![Desktop View](/assets/img/StreamYard/FeatureDump/blur-intensity.png){: .normal}

You fell for it.



## More Custom Layouts

This was a fun feature - I was tasked with increasing the custom layout limit from 8 to 16 only for paid users, as well as adding the new settings entry point button in the row, next to the add and edit buttons. I got to touch some backend and a lot of frontend. One of the things to bear in mind was downgrade - if a user created more than 8 custom layouts while subscribed, and then they cancelled their membership, the layouts should be greyed out, and upon click show an upgrade modal.

![Desktop View](/assets/img/StreamYard/FeatureDump/mcul.png){: .normal}

![Desktop View](/assets/img/StreamYard/FeatureDump/mcul-upgrade-modal.png){: .normal}

Also it was hard because given the increased amount of layouts, I had to consider how to show this for portrait and mobile, and test a lot (because that component is declared stupidly).

Then, I had some time and improved the animation to show and hide the extra layouts, which before was just a blink.

<iframe class="embed-video" loading="lazy" src="/assets/img/StreamYard/FeatureDump/mcul-animation.mp4" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen=""></iframe>


## Entrance Revamp

I worked on creating 2 new versions of the entrance studio, one wth the 3 controls and the other without, to test whether this affected the number of users actually going into a studio. It also looks better than the usual one.

![Desktop View](/assets/img/StreamYard/FeatureDump/entrance-revamp.png){: .normal}

vs the default:

![Desktop View](/assets/img/StreamYard/FeatureDump/old-entrance.png){: .normal}


## AI Clips Modal After Stream

This was actually my first task in the Product squad (my second squad). The goal was to add a modal when users finished a broadcast to raise awareness about the AI Clip generation feature - you get the highlights of your broadcast as short, shareable videos in portrait mode and with captions, kind of what you see on Instagram and TikTok all the time. Then with a few clicks, you can directly share them on your platforms.

![Desktop View](/assets/img/StreamYard/FeatureDump/ai-clips-modal.png){: .normal}

There were 2 main issues: first, AI clip generation only works for videos of lengths between 30 seconds and 2 hours, as this is what the backend model takes as input. However, it was not easy to calculate the length of a reusable studio broadcast, and they work with episodes whose lengths are cumulative from the first one. So for pragmatism, I reported this back and we decided to ignore that case for now as it impacted few of our target users.

Second, for AI clip generation the video must first have been processed, meaning that upon finishing a broadcast we must first wait for it to upload to the cloud, and then we can begin generating. Since this can take a while, and there was no proper backend handling of messages to know when processing had finished in that part of the app, I had to come up with a clever workaround under the time constraint. I figured that since upon closing the tab, any hooks would unmount and so we couldn't use a simple `useEffect`, I would instead open a second tab, which showed a modal saying "video processing, please don't close this tab" first, and upon completion showed "AI clips are generating, you can now leave this page". Since we had been super close to having to scratch the task altogether, my teammates were happy to at least be able to finish and ship it on time, even if it was suboptimal, and this was my first taste of just how much agency I can exercise as a Bending Spoons dev. And side note, since then, we have added the proper message handling to no longer need this!



## Referral Countdown

In an attempt to make referred users more aware of their available credits, I added a referral banner, with 2 versions, a simple one (just the banner), and an aggressive one (with a countdown).

![Desktop View](/assets/img/StreamYard/FeatureDump/simple.png){: .normal}

![Desktop View](/assets/img/StreamYard/FeatureDump/aggressive.png){: .normal}

But users _really_ didn't like the aggressive variant and we quickly had to close that down as it affected metrics negatively.


## Banner to Return to Dashboard in Mobile

This one really is simple. But before on mobile, we had no way for users to return to the dashboard once in the studio. So I just added the banner with the SY logo as we have in desktop.

![Desktop View](/assets/img/StreamYard/FeatureDump/banner-mobile.png){: .normal}


## New Guest in Studio Pop Up

As some users had asked for a better way to manage when guests enter the studio, ideally a shortcut to add them on stage, I was tasked with adding just this.

![Desktop View](/assets/img/StreamYard/FeatureDump/new-guests.png){: .normal}

Upon clicking this, it directly adds the user to stage, or it disappears after a few seconds if not clicked. It also handles the green room case to instead display "Add to studio".

Since I was touching the alert banners, I then also revamped the "mic is muted" one:

![Desktop View](/assets/img/StreamYard/FeatureDump/mic-revamp.png){: .normal}

And added settings to be able to disable these alerts altogether if they were dirsuptive to the user, and to hide the resolution tag.

![Desktop View](/assets/img/StreamYard/FeatureDump/settings.png){: .normal}

Also added some very minor UI changes to revamp the stage: border radius to the stage, some top margin, changed the opacity of the resolution tag, and moved the "Enhance my appearance" setting to the tab I renamed "Visual Effects" as it was not clear before.

![Desktop View](/assets/img/StreamYard/FeatureDump/revamped-stage.png){: .normal}


## Split Live and Recording

The default experience in StreamYard is that if you are in a recording studio but add a destination, you switch to a live stream studio, and vice versa if you delete your destinations. The types of studios are separate entities. We wanted to see if we could simplify the codebase by splitting these further so you can never switch between them. This task was very technically difficult and included a bunch of architecture and UI changes. In the end, metrics were slightly impacted negatively and we decided to keep the default experience.


## Revamp Feedback Modal


## Others

Other non-user-facing features include codebase cleanups, mainly of feature flags but also of support to competing platforms like Socio and ReStream, and refactoring our internal messaging system for video updates to use Zod instead of Joi for schemas and validations for better type inference. I also added a bunch of missing metrics
