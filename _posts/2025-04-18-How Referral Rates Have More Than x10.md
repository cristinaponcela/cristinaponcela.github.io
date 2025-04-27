---
title: "How Referral Rates Have More Than x10"
date: 2025-04-18 08:42:00 -500
categories: [StreamYard, A/B Testing, Referrals]
tags: [production, efficiency, my journey]
image: "https://raw.githubusercontent.com/cristinaponcela/cristinaponcela.github.io/refs/heads/main/assets/img/StreamYard/Referrals/banner-dana.png"
---

No matter how good a product's marketing is, word of mouth will always account for a significant part of customer base expansion. And the only thing better than having a valuable customer is have the customer bring in several other valuable customers. As such, we have been striving to introduce several features to make sharring referral links easy, and it is already reaping a great amount of success.

## A/B Tests

To figure out exactly where on the screen users are more likely to click to share a referral link, we ran a few A/B tests. 3 of them have been the most promising, out of which I was the dev for 1, so I can explain this one in more detail.

Our default tab in the studio, that is, the tab that first appears as open when a user enters the studio, is the Comments tab. It makes sense - our users mainly interact with comments, and do this more repeatedly throughout a stream (as setting brands, music, or showing assets is of course also popular, but is done once or twice per stream, unlike interacting with comments).

So, it was a natural choice to somehow allow the user to interact with referral links here. The initial design was to add a "Fill" button - something on top of the `CommentsInput` that would fill it with the referral link. Since the designer and product manager were skeptical that this would be easy to do knowing StreamYard's structure, the requirement was to try this, but if too hard or time-consuming just add a "Copy" button to copy the referral link to the clipboard so the user could then manually send it.

Not being satisfied with the added complexity of this flow for the user, I decided to dig a bit deeper, and found another design that had been scrapped - a button that allowed the user to directly send the referral link as a comment. I knew I could give the team a nice surprise, so I started working on it.


## Referral Link in Comments

I did have to change some of the logic for the hook that sends the comment, as it before only took a mouse event as input: the user clicks send, whatever is in the `CommentsInput` is sent. There was no variable or process to change the initial value. Something like:

```typescript
const onSubmit = useCallback(
  async (e, submitValue) => {
    // Use either the provided submitValue or fall back to the existing value
    // This is important because if the user starts typing but the clicks "Send"
    // for the referral link, we should not be concatenating the referral link
    // to the user's unfinished comment
    const textToSubmit = submitValue || value;
    
    // Show loading indicator
    setLoading(true);
    
    // Process submission to each selected destination
    const submissionRequests = selectedOutputs
      .map(output => {
        // Validate the submission for this destination
        const validationError = validateSubmission(output);
        
        if (validationError) {
          // Handle validation failure
          logFailedSubmission({
            output,
            text: textToSubmit,
            error: validationError
          });
          return null;
        }
        
        // Submit to this destination
        return submitToDestination({
          output,
          text: textToSubmit
        });
      })
    });
```

Since I added this second input, we now had the necessary parameters available to start the implementation. A simple `useCallback` hook was enough to handle this:

```typescript
const handleReferralLinkAction = useCallback(() => {
		setValue(referralMessage);
        onSubmit(null, referralMessage);
}, [referralMessage, onSubmit, setValue]);
```

And because it takes a second to load the comment and it looked weird loading and sending an empty `CommentsInput`, I also added `setValue` to the 'onSubmit' comments hook so we could populate the `CommentsInput` with the referral link in the UI. See below how it would look otherwise:

<iframe class="embed-video" loading="lazy" src="/assets/img/StreamYard/Referrals/referral-no-set-value.mp4" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen=""></iframe>

Vs. the final result:

<iframe class="embed-video" loading="lazy" src="/assets/img/StreamYard/Referrals/referral-link-in-comments.mp4" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen=""></iframe>

And the mobile version:

<iframe class="embed-video" loading="lazy" src="/assets/img/StreamYard/Referrals/referrals-mobile.mp4" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen=""></iframe>

Plus, of course, the necessary metrics to track the experiment's performance (which I also got to dabble in! Since I didn't want to wait for data's proposal on this one, I created my own and it was approved, so I was also in charge of not only adding but creating the metrics :D).


## Conclusion

Among the other 2 experiments, this one has contributed in increasing the number of users that sign up using a referral link by more than tenfold; from 1 referral every 200 segmented users before, to 6 referrals every 100 users now!

THis was a very fun task, especially considering I surpassed the requirements and made my bosses happy, and managed to finish the implementation and release everything in just 2 days. Another lesson learned with how to conduct A/B testing to make a difference, and see the difference just a few weeks after release.