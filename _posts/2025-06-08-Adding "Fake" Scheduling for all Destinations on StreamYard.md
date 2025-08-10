---
title: "Adding "Fake" Scheduling for all Destinations on StreamYard"
date: 2025-06-08 08:42:00 -500
categories: [StreamYard, A/B Testing, Destinations, Product Design]
tags: [production, feature flags, my journey, changing requirements]
image: "https://raw.githubusercontent.com/cristinaponcela/cristinaponcela.github.io/refs/heads/main/assets/img/StreamYard/FakeScheduling/scheduling.png"
---

As for any other product, at StreamYard we have a drop of users in our funnel between landing on the dashboard and actually starting a broadcast. We wanted to test a hypothesis - is this decline due to users being afraid to record or go live when they have newly joined, or is it because of something else?

## The Scheduling A/B Test

This is why we decided to test: if we provide an experience where scheduling is more prominent, will this increase the percentage of new users that start a broadcast? And will it have any change on the average billing per user (ABPU), as less users will be driven off of the product from that initial fear?

The initial design looked something like this:

![Desktop View](/assets/img/StreamYard/FakeScheduling/prominent-scheduling-modal.png){: .normal}

When you click on the "Enter the Studio" CTA, it follows the usual flow - if you don't have added destinations, you see the `AddDestinationModal`, and if you do you see the `CreateBroadcastModal`. Note that once you do add a destination from the `AddDestinationModal`, it immediately takes you to the `CreateBroadcastModal` with that destination added.

![Desktop View](/assets/img/StreamYard/FakeScheduling/AddDestinationModal.png){: .normal}

![Desktop View](/assets/img/StreamYard/FakeScheduling/CreateBroadcastModal.png){: .normal}


And if you click on the "Schedule for Later" CTA, it takes you through the same flow, with the difference that the state has `scheduled` true, and when you click "Create live stream" it schedules it instead of taking you into the studio. (And the title reads "Schedule live stream" instead of "Create live stream", which believe it or not was hard to implement because these are determined by a hook `useBroadcastTitles` that depend on dozens of conditions for several cases.)

The main difficulty of this is that sometimes StreamYard code, especially in the critical components, is whacky af. This is because we acquired the product about a year and a half ago, and refactored most things, but critical components were always more delicate and have very ✨interesting✨ logic and software design decisions, so it would be easier to refactor from scratch, and this would take too long.

For example, to add the modal into the flow, I added something like:

```typescript
const initialPage = useMemo(() => {
		const currentPage = modalSelectors.getPage(state);

		if (
			featureFlags.prominentScheduling &&
			currentPage === '' &&
			broadcastContentType === 'livestream' &&
			!hasSeenProminentSchedulingModal &&
			!broadcast // never in the studio
		) {
			return 'prominent-scheduling';
		}

		// If no destinations have been connected, 'add-destination' page is shown
		if (
			currentPage === '' &&
			filteredDestinations.length === 0 &&
			broadcastContentType === 'livestream' &&
			!hasSkippedDestinationSelectionInDestinationsPage
		) {
			return 'add-destination';
		}
		return currentPage; // currentPage === '' represents the CreateBroadcastModal, and it is the default
	}, [
		state,
		filteredDestinations.length,
		broadcastContentType,
		hasSkippedDestinationSelectionInDestinationsPage,
		hasSeenProminentSchedulingModal,
		broadcast,
	]);
```

Then calling the `ProminentSchedulingModal` at this page. It looks like:


![Desktop View](/assets/img/StreamYard/FakeScheduling/scheduled-broadcast.png){: .normal}


There was a slight problem with this task though - before, StreamYard had only implemented scheduling for YouTube, Facebook and LinkedIn, and this is done through their respective APIs, which allow you to create the "post" on the platform so it can be found before the Live date and time.

Therefore, this design was suboptimal. If we click on "Schedule for Later", shouldn't we only see those 3 destinations? The task was redundant if this was the case, as this provides an inferior UX to the user than the default experience, which is always an immediate strike out for any task.


## What Do I Mean by "Fake" Scheduling?

I spoke to my PM and team leader about this issue, and since a solution was not immediately obvious, they told me they would rethink the task. I then came up with an idea - I'll just make scheduling possible for all destinations. It was not "true" scheduling, as there was no API implementation for other platforms, so no post on the platform is created beforehand. But destinations may not even _have_ this option or a public API, and this amount of backend work is usually overkill for an A/B test. 

Thus, I decided to at least find a way to allow users to schedule live streams and see them in their StreamYard dashboard. This functionality is definitely still useful at an organizational level, and for sure would achieve the goal of the task anyway of taking away that initial fear by allowing users to firstly schedule live streams.

Plus, even when you do schedule a live stream to one of the destinations with API on StreamYard, it still doesn't go live automatically. You still have to come to the studio at that time and go live. So the only difference really between these "fake" and "true" scheduling options is the fact that the post is visible on the platform before the Live time and date, and weeks of dev time.

Something I have had to learn in this role is pragmatism - if you have a task for an A/B test because you want to test a theory of user behavior, it is not worth it to spend weeks adding backend code that may have to be deleted or have very little benefits if users don't receive the feature well. It is always best to work incrementally, and if the test succeeds, then add the full implementation.

So it was decided - I had 24 hours to figure out how to add "fake" scheduling to all destinations before the task was potentially discarded.



## Adding Scheduling for All Destinations

After investigating how scheduling worked for our 3 supporting destinations, it turned out it wasn't all that difficult to add. We have helper functions to create and update the outputs, and these are each platform specific. In the payload, YouTube, Facebook and LinkedIn had the property `plannedStartTime`, which when read by the frontend would declare the state as `scheduled` is `true`. So all I had to do was add this value to all payloads.

Then it was a matter of correctly hiding the "Schedule for Later" checkbox, because if we click on this CTA in the modal we clearly don't want to have to re-select it in the `CreateBroadcastModal`. This actually took some time to find, because we initialize the `scheduled` value in state using a helper function `getNewOutputInitialValues` that is _very_ big and kind of obscured in the codebase. Then it was easy enough to just set:

```typescript
scheduled:
    wasTriggeredBy === 'prominent-schedule'
        ? true
        : getInitialScheduledValue(...),
```

And just like that, all destinations allowed scheduling!

![Desktop View](/assets/img/StreamYard/FakeScheduling/all-destination-scheduling.png){: .normal}

![Desktop View](/assets/img/StreamYard/FakeScheduling/all-destinations-scheduled.png){: .normal}

## Other Requirements

Other than this, I just needed to implement small UI changes. First, adding the `ScheduledTag` to the top right of the studio for scheduled live streams, to show the date and time that they are scheduled for:

![Desktop View](/assets/img/StreamYard/FakeScheduling/scheduled-studio-tag.png){: .normal}


And then adding a CTA in the studio with the same UX and state management as clicking on the "Schedule for Later" CTA in the modal - it hides the `ScheduleCheckbox` and sets `scheduled` to true.

![Desktop View](/assets/img/StreamYard/FakeScheduling/schedule-studio-cta.png){: .normal}


And that was that! I could have a call with my colleagues the next day and give them a very happy surprise :)


## Conclusion

I particularly liked this task because when there is a problem with a task's requirements, I try to take it as an opportunity to think outside of the box, find a solution and re-shape the task, and I'm happy that I work in a place where I can do that. I also find that taking extra time to investigate tasks' feasibility, and taking agency and responsibility for your tasks to find ways to bring them to the finish line usually ends in happy teammates and success stories.

In the end, the test was quite successful - we found out that late stage ABPU increased by 40%, and +20% users chose to schedule a broadcast, suggesting pent-up demand for more scheduling visibility. However, -4% users entered the studio and -3% users creating a broadcast, suggesting having two options and an extra step to enter the studio has a damaging effect. Out of the segmented users and interactions with the `ProminentSchedulingModal`, 20% chose to schedule and 80% chose to enter now.

This was kind of inconclusive because we still need to investigate the drop, but the findings proved invaluable and achieved the goal of the task, which was to test a hypothesis on user behavior and what they want. For sure this has taught me that these kinds of experiments are more valuable than they seem at face value because they teach you in which direction to take your product given your customer base's behavioral patterns.