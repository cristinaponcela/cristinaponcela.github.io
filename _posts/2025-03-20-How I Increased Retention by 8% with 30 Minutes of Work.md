---
title: "How I Increased Retention by 8% with 30 Minutes of Work"
date: 2025-03-20 08:42:00 -500
categories: [ StreamYard, A/B Testing]
tags: [production, efficiency, my journey]
image: "https://raw.githubusercontent.com/cristinaponcela/cristinaponcela.github.io/refs/heads/main/assets/img/StreamYard/AddDestinationButton/AddDestinationButtons.png"
---

## How I Increased Retention by 8% with 30 Minutes of Work

Last time, I spoke about the two new approaches we tried to decide which functionality additions to prioritize:

1. Find your weakest metrics, and find a way to improve user activation in this area,
2. Find your strongest metric, and reduce user friction in that area,

And I gave an example of the first one. Today, I want to share an example of how I used the second one, and how 30 minutes of work here increased user activation with destinations by 5% and retention by 8%, our most crucial metrics.


## Context

In StreamYard, the main focus is to encourage users to start broadcasts (our product’s main functionality), ideally with destinations - this means that, instead of just recording something locally, they stream it to a destination, like YouTube, Twitch, Facebook, and so on. This makes sense; if a user goes live with destinations, it will reach more users and therefore potential customers. We particularly value users who start streams with destinations that are longer than 10 minutes, and this is what we mean by “user activation with destinations”.

Obviously, we have metrics for the parts of the flow where users add destinations and go live, and as you can imagine these are some of the strongest in the application. Which is exactly why we decided to look into it - if we have something that users love and works well, and if we can make it better, the benefit will be great. Though this, of course, is also a risky decision, because if you fail and the experience is worse, you just hurt the main experience in the product. 

Thus, I was tasked with a double-edged task: make adding destinations in the studio easier for users. And that is what I did, but to deal with the previously mentioned risk, I did so under a feature flag to test on a small percentage of users first.


## Past vs Present

The past experience was:

![Desktop View](/assets/img/StreamYard/AddDestinationButton/old-add-destinations-step-1.png){: .normal}

You clicked on Record (or “Registra, I have it set to Italian lol).

![Desktop View](/assets/img/StreamYard/AddDestinationButton/old-add-destinations-step-2.png){: .normal}

And then had to click on the plus sign, which only upon hovering would let you know that is where you can add destinations. Not the most clear.


I added the following `AddDestinationsButton` at the top:

![Desktop View](/assets/img/StreamYard/AddDestinationButton/new-add-destinations.png){: .normal}


And as you can imagine, this change can’t have taken more than 30 minutes. But the impact was great, so much so that this will soon become the default experience.


## Conclusion

By leveraging the power of feature flags for testing on small groups of users first to determine effectiveness, and by implementing some well thought-out and justified small changes to the main parts of the application flow, you can achieve some very easy but large scale impacts. And just as short and sweet, this article ends here.
