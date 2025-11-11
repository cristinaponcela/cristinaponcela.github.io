---
title: "A/B Testing Can Also Be Used To Reduce Costs"
date: 2025-08-17 08:42:00 -500
categories: [StreamYard, A/B Testing, QoX]
tags: [production, costs, my journey]
image: "https://raw.githubusercontent.com/cristinaponcela/cristinaponcela.github.io/refs/heads/main/assets/img/StreamYard/NPS/nps.png"
---

Usually, most of our A/B tests are run to implement a new feature and test its adoption among users. However, at the Quality of Experience (QoX) squad, we have gotten increasingly creative with how we use our feature flags.

As opposed to other squads, in QoX we add a feature flag to every single task. This is for two main reasons: to be able to quickly go back to the default experience if errors arise, since we have the ambitious deadline of releasing 2 features every single week so there is not always enough time for proper testing, and to switch the feature flag on to show the new feature to all users at a specific point in time. Every Monday and Thursday at 6pm CET, we release a feature to all users with an announcement modal. Something to say, hey, look what's new!

![Desktop View](/assets/img/StreamYard/CloudRecordToggle/announcement-modal.png){: .normal}


This of course has a cost to the codebase in terms of complexity and maintainability, as logic can branch into multiple variants, and old feature flags have to be routinely cleaned up. But it has the added value that users now know when to expect new features and are hyped about it - we have seen a massive surge in the product's perception within the community.

Most recently, we decided to try yet a new use for a feature flag - can we A/B test cost reduction?


## Overview

The idea was simple: on StreamYard, paid users have cloud recording enabled by default for all live streams and recordings. While there is a storage limit in terms of hours (which also depends on your plan's tier), we realized most paid users have a bunch of hours of recording, though not at the limit, even though they never interact with said recording. So surely adding a toggle to allow paid users to decide whether or not to record their live streams could save us some cash.

But also, for free users the CTA to upgrade when going live to unlock cloud recordings was super hidden. 

![Desktop View](/assets/img/StreamYard/CloudRecordToggle/hidden-cta.png){: .normal}

Yes, that tiny link is all that lets free users know that they can unlock cloud recording for live streams, and takes them to the page to upgrade. So the second motivation for this task was to test whether a more prominent toggle, which for free users triggers an update modal, would lead to an increase in signups.


## Implementation

### A/B Test Configuration

The test was implemented using a feature flag, lets call it `showCloudRecordingToggle`. It had 3 states to gain full insights from the new experience: `control` (keeps the default experience, no toggle and is recorded for paid users), `true` (shows the toggle as on by default, except for free users, records unless toggled), `false` (shows the toggle as off by default, doesn't record unless toggled).

Problem is, adding a new condition on a feature this delicate and that touches so many parts of the codebase wasn't exactly trivial. Firstly, I updated the backend `createBroadcast` logic to initialize this value. We also store the preference in the workspace object - if a user toggled from on to off, we don't want to make them repeat this action on every new broadcast, we should now default to off.

```typescript
// Backend creation logic
const workspaceFlags = await getFeatureFlags(team, server);
const firstDefault = workspaceFlags.showCloudRecordingToggle === 'true';
const cloudRecordingPreference = workspace.cloudRecordingPreference ?? firstDefault;
```

### Conditions

Users in the control group (`showCloudRecordingToggle === 'control'`) maintained the original behavior:
```typescript
const defaultRecord =
    isRecordOnly || // free users also have access to cloud recording, even if typically restricted to 5 hours
    isPrerecordedBroadcast || // skip this case as also always allows cloud recording
    isWebinar || // this toggle was not supposed to be included in webinars, which btw are already only for paid users
    workspaceFlags?.showCloudRecordingToggle === 'control';
```

Then I had to hide the toggle in the UI for the default experience (control).
```typescript
const shouldShowCloudRecordingToggle =
    workspaceFlags.showCloudRecordingToggle !== 'control' &&
    shouldCloudRecord !== undefined && // we then store the value in the broadcast object
    allOutputs.length > 0 && // same as isRecordOnly
    !isPersistentBroadcast &&
    broadcastType !== 'prerecorded' &&
    !isWebinar;
```

And then we also depend on `canTeamRecord`, which we only had in the backend and I had to create a selector for in the frontend copying the logic. Essentially, this returns true if the user is free, it is record only, and has time left in their free 5 hours (which may be more if another feature flag is active), or if the user is paid, they have enough hours left of their increased storage limit, or they have auto delete enabled. 

But also, we have 2 features, local isolated recordings and audio recordings, which we offer to paid users and depend on cloud recording. So I also had to update the handlers to turn these off is cloud recording was toggled off, but also to toggle cloud recording on if either of the features were toggled on. To handle dependencies appropriately:
```typescript
// When disabling cloud recording, we must also disable local recordings
if (newValue === false) {
    if (localIsolatedRecordings) {
        updateValue({
            shouldCloudRecording: false,
            localIsolatedRecordings: null,
        });
    }
    if (shouldRecordIndividualAudio) {
        updateValue({
            shouldRecordIndividualAudio: false,
        });
    }
}
... // and vice versa
```

So there were easily 10 conditions I had to carefully pass around the codebase, add access to in certain backend files by adding values to the payload, and ensure the experience was correct in all cases. This was not as trivial of a task as initially expected xD.

## UI

The UI was as follows:

In the header, you can see the toggle, and then again in the go live modal for extra visibility:

![Desktop View](/assets/img/StreamYard/CloudRecordToggle/cloud-record-toggle.png){: .normal}

And in the case that a paid user runs out of the storage hours available on their plan, a modal to encourage them to purchase more or delete other recordings to make space:

![Desktop View](/assets/img/StreamYard/CloudRecordToggle/more-hours.png){: .normal}


For free users, this triggers the upgrade modal:

<iframe class="embed-video" loading="lazy" src="/assets/img/StreamYard/CloudRecordToggle/upgrade-modal.mp4" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen=""></iframe>

The mobile UI:

![Desktop View](/assets/img/StreamYard/CloudRecordToggle/record-toggle-mobile.png){: .normal}

I then added the metrics to track user interaction with the toggle. 


## Results

Unfortunately, and as we kind of expected, within the first few days many users in the `'false'` segment opened tickets to complain. Not because there was any bug, but because they had not noticed the new toggle in the studio header and were upset that they lost their recordings. Also, another team is currently testing a new feature that has a strong value proposition but depends on cloud recording, so for now we have closed the experiment and delayed it to be leveraged at a later stage. But we expect substantial cost reductions if it is adopted well, and if next time we communicate it to existing users properly.


## Conclusion

The cloud record toggle A/B test demonstrates how thoughtful UX changes can simultaneously improve cost efficiency and revenue opportunities. I am looking forward to reopening the experiment and coming back with some more insights!
