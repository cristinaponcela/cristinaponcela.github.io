---
title: "Conducting User Research Within Your Platform"
date: 2025-06-26 08:42:00 -500
categories: [StreamYard, User Research, QoX]
tags: [production, data, user interviews]
image: "https://raw.githubusercontent.com/cristinaponcela/cristinaponcela.github.io/refs/heads/main/assets/img/StreamYard/EntryPoints/survey.png"
---

In my current squad, User Acquisition (UA), we ship features to improve the product and run tests to measure how this impacts signups and ABPU. A lot of this is trying to understand drops in the funnel - if we have 100 users landing on the dashboard, but only 80 click on "Create a Broadcast", only 60 actually enter the studio, and only 30 actually go live, can we find the friction points to reduce these drops? 

To do this, we decided to go straight to the source; we asked users themselves what they think of StreamYard, how they use it, if they had any difficulties with the product, and importantly which features they would like to see and would help further justify the price.


## User Interview Entry Points

I added 3 main entry points for users to be complete a Typeform survey, whose last step offered them to book an interview with us. Initially, we offered them a coupon in exchange, but soon we realized we had an influx of users who were happy to do this regardless of the incentive.

1. In the dashboard:

I added some logic so that, after release, once the user landed on the dashboard after exiting the studio (which is typically done after finishing a broadcast) for the third time, a modal would appear. If they dismissed it, it would reappear on the 6th time. If they took the survey, it wouldn't appear again.

![Desktop View](/assets/img/StreamYard/EntryPoints/dashboard-entry-point.png){: .normal}


2. On downgrade:

When a paid user decided to cancel their subscription, or downgrade it to a cheaper plan, we wanted to gain insights into 3 main reasons for users may have chosen for cancelling:

- I didn't understand how to use the product.
- Missing features I need.
- Switching to another product. 

These were set as 1, 2 and 3. Since we only show the interview entry point when one of these is selected (and if multiple are selected, we show it in the first one clicked. And if unselected, we move it to the next one, etc), it is not possible for the user to access the Typeform server without having at least one of these selected. Which means we could then append the reasons selected (e.g 1, 23, etc) nad the user email to the Typeform URL, for us to process on our end.

![Desktop View](/assets/img/StreamYard/EntryPoints/downgrade-entry-point.png){: .normal}


3. On account deletion:

I updated the flow to include a step for users to be able to take the survey on why they are leaving the product, which is also invaluable for us to know.

<iframe class="embed-video" loading="lazy" src="/assets/img/StreamYard/EntryPoints/deletion-entry-point.mp4" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen=""></iframe>


## User Interviews

So far, at least 20 user interviews have been conducted by the team, some of those by myself. This has been extremely fun and fulfilling - nothing makes you as proud of your work as having an interview with users who keep going on about how much they love and use StreamYard's latest feature, which you built.

My favorite one so far was with a pair of users that had prepared a list of possible features they would love to see. This was pure gold for our backlog, and they were lovely all around.

In another interview, the user had signed up on the same day he had deleted his account. He was looking for a way to do a webinar, and mentioned that our closing CTAs for modals were not clear enough. Since he couldn't manage to follow the full flow, he deleted his account.

These kinds of insights, whether big or small, help us improve the product immensely, plus they make the users who participate in them rightfully feel like they are contributing to the product and building something with us, further improving brand perception. Especially if your product depends on few high-fidelity paying subscribers, this is more of a win-win.


## Other Ways in Which We Conduct User Research Within the Platform

We have an attribution survey for new users as a first step when they sign up, to find out how they heard about the product. I recently added an option "AI recommended it" because yes, that's now a thing that can happen.

![Desktop View](/assets/img/StreamYard/EntryPoints/attribution-survey.png){: .normal}


Also, we use Zendesk to process user tickets. I recently added a tag, "new user", to those tickets created within the first 60 minutes after user signup. This is particularly helpful to find out friction points and explain drops in our funnel.


## Conclusion

While most of our user research comes from A/B testing and metrics, you don't need a huge amount of hours and fancy tests to get to know your users and how they interact with your platform. Sometimes taking it back to the source can be equally as helpful.
