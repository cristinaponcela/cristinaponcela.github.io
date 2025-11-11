---
title: "Completing the Puzzle of a Stupid API"
date: 2025-09-03 08:42:00 -500
categories: [StreamYard, APIs, Zendesk, QoX]
tags: [workarounds, my journey, creativity, problem solving]
image: "https://raw.githubusercontent.com/cristinaponcela/cristinaponcela.github.io/refs/heads/main/assets/img/StreamYard/Zendesk/ZENDESK_API.png"
---


You know when you get a task and you shrug to yourself and think "sure, easy enough". And then it ends up being the most unnecessary, wild and fun experience ever. That is my experience every time I come into contact with the Zendesk API lol. This one was no exception.


## The Task

As a small improvement to make users access their tickets more easily, we had recently added a new option in the dashboard user menu that read "My support tickets", and upon clicking it would take you to your Streamyard Zendesk portal. The follow-up task seemed easy at first glance: add a notification dot if the user has unread replies or other updates to one of their tickets.

IMAGE OF REQ

The thing is, on Zendesk's side (once you leave SY), you see a list of your tickets, and the unread ones have boldened titles. So from a Product perspective, it seemed feasible enough that we should be able to access such a property and render a component on the SY dashboard accordingly. Nope.

IMAGE OF BOLD TITLES

What was initially estimated as a 30 minute task quickly became a task with a dev estimate of 2-3 days, since we would somehow have to find a way to make the correct API calls to obtain the information we needed.


## Why it was Unnecessarily Difficult

Here's the thing; as you can see in the [Zendesk docu](https://developer.zendesk.com/api-reference/ticketing/tickets/ticket-requests/#list-requests) for requests, they have no way to request the user id that they use on their side.

When a user first clicks on the menu to access Zendesk, Zendesk creates a user id for them based on the user id we have on our side. Obviously these can;t match, which makes sense given how many platforms Zendesk supports - surely they would have duplicates if they didn't handle this appropriately. The authenticated request is made by creating a JWT token. However, such a token can not be used to make user-specific API calls. We can only do so by using our env token as the bearer, and request to an endpoint containing the user id.

However, all user-specific calls require the Zendesk user id. So okay, we need to find a way to obtain a user's Zendesk id from their StreamYard id.

Also, Zendesk doesn't have a "last read" value in any of its API responses. So the closest thing we can do to achieve the desired behavior is to call the endpoint `https://streamyard.zendesk.com/api/v2/users/${uzendeskUerId}/requests.json`, and filter by `updated_at`. But this means we have to somehow know when the user last checked their tickets, so we can compare the timestamps to decide whether the specific update has been read or not. And guess what, Zendesk also doesn't provide a way for us to know when the user enters their SY tickets dashboard.

So in summary, we need 2 things we don't have access to before we can complete this puzzle: the Zendesk user id, and the timestamp of when the user last landed on the Zendesk requests dashboard.


## The Lightbulb Moment

Something I have had to learn this year is to exercise pragmatism when needed. It was quickly obvious that there was no way for us to have the necessary information to deterministically know, from our side on the StreamYard dashboard, whether a user had unread ticket updates. So the question now was how close I could get to the desired behavior with a creative solution.

There is only one place in which we ever have access to the Zendesk user id, also know as the `requester_id`, and where this is included in a useful API response. And that is when we query for all tickets when processing them in a cron job: so we make a call to `https://streamyard.zendesk.com/api/v2/search.json?query=${encodeURIComponent(query)}`, with a query like `type:ticket status:open status:pending status:new`, and we receive all tickets with the full json object response, from which we can access the Zendesk user id. And at this stage, we had access to the StreamYard user id. So it was as easy as adding the Zendesk user id to the user object when the tickets are processed, for each user.

Also, while we can't know when a user lands on the Zendesk dashboard, we _can_ know when they leave from StreamYard to access Zendesk. So each time this was done, we record `lastCheckedZendeskTickets` timestamp to the user object.

So now, we could make the request using the Zendesk user id, fetch all the `updated_at` for all of a user's tickets, and compare these to `lastCheckedZendeskTickets`. if at least one timestamp is newer, we show the notification dot.


## Limitations, Mitigations and Justifications

This solution is not perfect. Note that there are several assumptions that we make that are not always true.

1. A user leaving from SY to Zendesk is the correct `lastCheckedZendeskTickets`.

This is not true, since when a user gets a ticket update, we also send them an email. So in the case where a user clicks on the link from their email to check the update (or in some other way), we don't have access to either platform to know of this. Hence, we can have a situation where a user sees the notification dot in their menu, but when they click it there are no actual unread updates, since they already checked them.

However, this case is not significant - we assume that if a user is clicking on the email links to check their emails, this is likely their main source of information regarding Zendesk, and hence they place little value on the menu option within the dashboard. And even if not the case, this is not a disruptive state.


2. We always have access to the user's Zendesk user id.

Note that we only store this in the cron job that processes new tickets. This means that only tickets created after the release would be processed such that the code to store the Zendesk user id is accessed. So if a user created tickets only before the release, we can never access their Zendesk user id, thus the request returns `null`, and we cannot show the notification dot accordingly.

But again, this is negligible. Tickets are usually solved within the same day, so we assume that after 1 or 2 weeks all old tickets are irrelevant. So no need to show updates since there aren't any.


3. We assume that we can access `lastCheckedZendeskTickets`.

Again, this value is only populated if the user clicks on the menu from our dashboard. In the case that they never do this, we can never compare the timestamps to decide the state. To encourage users to click it and initialize this value, if `!lastCheckedZendeskTickets`, we show the notification dot by default. However, we only do so if `response.mostRecentUpdate` is `!null` - this means we have access to the user's Zendesk user id, made the request, and they indeed have at least 1 ticket (so 1 instance of `updated_at`). In fact, the response of the call to the endpoint is just 1 `updated_at`, that of the most recent one. 

```typescript
if (!mostRecentUpdate || request.updated_at > mostRecentUpdate) {
						mostRecentUpdate = request.updated_at;
					}
```

This is because we only need to know if *at least* one ticket update is unread, so the comparison with the most recent timestamp is enough.

Here, we can once more fall into an inconsistent state, if the user never clicked `My support tickets` in the menu, but they did create a ticket after the release and checked its updates via email. They would see the notification dot but have no updates.

In this case, we can also rest assured - this is quite corner case, and since the menu option itself is a new addition it can be interpreted as "oh, the dot is there to show a new feature". Either way, we'd rather have the data populated for all future calls, and again this is not disruptive.

```typescript
if (!isCurrentRequest) return;

			if (!response.mostRecentUpdate) {
				setHasUnread(false);
				return;
			}

			if (!lastCheckedSupportTickets) {
				setHasUnread(true);
				return;
			}

...

setHasUnread(mostRecentDate > lastCheckedDate);
```

So, in order, these checks ensure that:

1. We avoid race conditions.
2. We don't bother showing the notification dot if the user hasn't created new tickets after the release (they have no need to check Zendesk),
3. but if they have and haven't used the menu, we encourage them to populate the data, even under the possibility of an inconsistent state.


## Conclusion

This task was SO fun. It really did feel like having to find all the pieces for a puzzle through detective work. By adding this implementation, I managed to add the feature as close as possible to the deterministic behavior intended, in what I think was a pretty clean and smart way. It took a lot of docu investigation and staring into a wall, but the result was very satisfying, and in the vast majority of our use cases, the state is consistent. And no-one has complained, only reported that it works pretty well, so that's always a plus!
