---
title: "How Small Changes Boost Revenue"
date: 2025-03-03 08:42:00 -500
categories: [ StreamYard, A/B Testing]
tags: [production, efficiency, my journey]
image: "https://raw.githubusercontent.com/cristinaponcela/cristinaponcela.github.io/refs/heads/main/assets/img/StreamYard/ai-clips/ai-announcement-modal.png"
---

## How Small Changes Boost Revenue

Leveraging A/B testing, making your product more comprehensible, or adding new functionalities can boost revenue by attracting new users or converting old users to paying ones. However, most of these efforts can take a long amount of time to implement, and are never guaranteed to have an impact. And as I have found out with experience, sometimes the smallest changes reap the greatest benefits.

I work in one of the Product squads at Bending Spoons, meaning I am in charge of bringing new functionalities to users. Most of the time, these functionalities range in size and impact, from long projects for a new functionality for all users to very small changes under a feature flag with a small testing group. Each time, we add the relevant metrics so we can assess the impact on users, and their acceptance of the change. We recently tried two new approaches to decide on which functionality additions to prioritize, and I will now live by these religiously:

1. Find your weakest metrics, and find a way to improve user activation in this area.
2. Find your strongest metric, and reduce user friction in that area.

Both of these have proven to show a very positive boost in metrics, and I will explain an in-detail example for case 1.


## The AI Clips Experiment

At StreamYard, we offer paying users the possibility to generate clips from their broadcasts using AI - you record your stream, click one button, and you have vertical and captioned highlights to go viral on platforms like Instagram or TikTok. And you can post them directly with one click too!

However, until recently ;), this functionality didn’t seem to be widely used by users. We weren’t sure whether this was because it was too hard to interact with (maybe users couldnt find it, or the UI to trigger generation wasn’t clear enough), or the quality of the clips was just not good enough.

As it happened in our case, the answer turned out to be both. The great guys at AI were tasked with creating a new version of the AI model - the previous model was quite faulty when finding the highlights of the video, would cut them in weird places, and required a strict Q&A type of conversation to work at even an okay-ish level. So they built v2.

Now my job was to find how to fix the other issue, and make the feature super easy to use and well-known by users. To do this, I ran 2 main experiments.

The first one was a feature flag that encapsulated the new version of the AI model (we still needed to test it with a controlled group of users first) and the _smallest_ changes I’ve ever added to an experiment: I added a tooltip when the videos were generating, to let the user know they would take tops 30 minutes to finish generating, and added an event in the backend procedure to get picked up by our emailing service to notify the user when the video generation finished. That was it!

![Desktop View](/assets/img/StreamYard/ai-clips/streamyard-generate-ai-email-tooltip.png){: .normal}

![Desktop View](/assets/img/StreamYard/ai-clips/ai-clips-email.png){: .normal}


For the second experiment, I added a modal at the end of broadcasts to push users to generate AI clips from the broadcast they just finished. This was mainly to show users about the functionality, but also to make its use easier - when you came to the Library page (where the AI clip generation can be triggered) from the modal, the generation would automatically be triggered. To do this, I added a query parameter ,something like `?autoAIClipsGeneration=true`, to trigger this, and then it gets cleared when the generation starts.

![Desktop View](/assets/img/StreamYard/ai-clips/ai-clips-modal.png){: .normal}

![Desktop View](/assets/img/StreamYard/ai-clips/ai-reday-to-generate.png){: .normal}
(Here, Generate is automatically “clicked” if you come from the modal’s CTA.)


### Some Challenges

A few small technical issues with this, that I will only get into because I’m a geek:

The way the backend is built, the AI generation can only be triggered when the broadcast’s video has been “processed”, meaning it has been uploaded to the cloud. When you finish a broadcast, processing is triggered first, so we needed to find a way to trigger generation right after processing finished.

The obvious initial answer is `useEffect` - let’s just wait until processing finishes, and when it does the change in state will get picked up and trigger the generation. This was actually the initial requirement. The problem was that they hadn’t realized that this crucial step of processing was an obstacle that couldn’t be easily skipped. If we did use `useEffect`, the generation would only be triggered if the user stayed on the page until processing finished, unlikely as this can take a few minutes, and more than suboptimal since the goal was to make interaction with the feature _easier_. Otherwise, the hook unmounts and the function is never triggered. 

So I tried to find a way around this - adding the flow to the backend procedure, having a poller to check the change in state and trigger it from the backend, or even a cron job. But there is a lot of pushback with adding big changes to the backend for experiments - we tend to avoid adding code complexity when we are not certain the feature will become a default. And since the modal was only supposed to teach users about the functionality and how to use it until the information was widely spread, this was definitely not going to make the cut. 



### My Creative Problem Solving Enters the Chat

Thus, I had to go to my Product Manager and tell him the news - we may have to adapt the requirements, or find a different solution. After a lot of back and forth with other members of the team, and when they seemed a bit hopeless with achieving a similar behavior to the originally intended, I came up with an idea. I would add the initial logic with a hook, but open the Library page in a separate tab (`_blank`) when the user clicked on the AI Clips modal. Then, I would add a non-dismissible modal, telling the user they couldn’t close the tab until the video finished processing, and that they could keep doing their thing on StreamYard from the initial tab. Then, when processing completed, another modal would show telling the user that generation had started and they were now free to fool around. Suboptimal, but a good solution to complete a promising experiment that was about to be scratched. I can assure you my boss was over the moon. 

![Desktop View](/assets/img/StreamYard/ai-clips/ai-processing.png){: .normal}

![Desktop View](/assets/img/StreamYard/ai-clips/ai-generating.png){: .normal}

And all was good and well, and we released both experiments.


## Results

As you can see, not major changes. Both experiments were pretty minor from Product’s side (kudos to AI for the new model, no small feat. 

Shockingly in comparison to the effort’s size, these changes had an excellent result: 

Higher adoption among users:
19.7% of subscribed users, and 16.6% of converted users, completing a broadcast tapped "Generate AI Clips".

The modal significantly increased interaction:
Tap on "Generate Clips": 5x higher for all users and 7x higher for converted users.
AI Clips downloads: 4x higher for all users and converted users.

The lesson I learned here: sometimes, the smallest changes, when well placed and with a clear impact in mind, can have the biggest outcomes.
