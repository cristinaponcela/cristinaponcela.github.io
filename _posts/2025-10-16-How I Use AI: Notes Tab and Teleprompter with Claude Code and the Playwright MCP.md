---
title: "How I use AI: Notes Tab and Teleprompter with Claude Code and the Playwright MCP"
date: 2025-10-16 08:42:00 -500
categories: [StreamYard, AI, MCPs, QoX]
tags: [production, claude code, efficiency, my journey]
image: "https://raw.githubusercontent.com/cristinaponcela/cristinaponcela.github.io/refs/heads/main/assets/img/StreamYard/Notes/"
---

The new challenge in software engineering is: can I prompt AI well enough, and give it enough context via MCPs or otherwise, that it can autonomously ship bug-free tasks?

I have been testing this out recently. In short, the current answer is no: if the task has any complexity, or your codebase is big, complex or has many in-house components, it is not yet possible to fully vibe code a task (or at least, not in a way that is meaningful. You can tell it exactly what lines to change, but that's not useful).

But I have been trying my best lately. And while not fully capable of delivering, AI can for sure do 80% of the workload, streamline the task and only require little human input.


## How I Use AI

As most developers, I mainly use Claude Code, and I usually prompt it in the following way:

1. A gain-context prompt.

This is where the AI gains its initial information of the part of the codebase we are going to modify, how the in-house components and the logic works, and probably the most critical step. It would be something like "I want to build ..., and add the file in the dir ... Please check component x", and ideally, if we have existing components or features with a similar logic, UI or structure, we make it analyze that too. The idea is to make the AI repeat back to us what it understood, so we can check it's on the right track.

2. Correct its assumptions.

Step 1 is only effective if you already have knowledge of the flow of that part of the codebase, and you can correct the AI's response. If it is missing anything or made a mistake in its explanation, you should reply with the correct explanation.

3. Make a plan.

Tell the AI to make a step-by-step plan of how it would implement said feature, *without* recommending any code yet. Bonus points if you already know exactly how to implement it and you give it the detailed plan yourself, it is obviously much more performant. But it still works well either way. And again, correct anything you don't agree with.

4. Prompt each step.

When you make a request that is too broad, Claude gets confused and misses many details. So I use 1 prompt for each step of the plan, and any subsequent prompts needed to perfect each step before moving onto the next one. It is important to do this to keep the conversation structured, otherwise it will gain context too early on other parts of the codebase and either hallucinate (e.g. call frontend functions in the backend), or produce nonsensical responses.

5. Go to Cursor to refine.

Once the main implementation is done, I find that Claude Code is quite bad in building UIs by itself, and especially in refining them, no matter how detailed the prompt is (e.g. "create a row with button such and such, padding x, etc" still fails to produce a decent result D: ). I follow a 3-try rule: if by the 3rd prompt to implement the same part of the feature we have not added any value, I switch to Cursor (and vice versa). Cursor seems to be much better at building specific things, especially since the open tabs you have at the time of prompt are its context. So I tend to have the in-house components that I want to use open, and tell it exactly how I want the UI built, and provide a picture of the Figma. By iterating on this enough, you can get a pixel-perfect result with very little human intervention.


*Fun fact:* if I spend more than 10 minutes back and forth between Claude Code and Cursor and feel like the progress to get closer to the design is too slow, I sometimes switch to [claude.ai](https://claude.ai/new) and try my luck there. Even though it lacks the codebase context, for generic functions it is quite successful where Claude Code and Cursor fail. For example, when I was working on adding the Notes tab, I needed to build a text editor from scratch. Claude Code and Cursor were great in creating the tab, perfecting the UI, correctly adding `notes` to the broadcast object and creating the selector to get and update this value on debounce, and adding this value as `nonIndexed`. However, the actual text editor was quite buggy; I needed to properly handle some complex states where, for example, some text is bold, we highlight that text plus more unboldened one, click bold, and it should make all of it bold. Also bullet point indenting upon typing `•` and `''`. I iterated a bit to fix these bugs to no avail. So I moved onto `claude.ai`, and boom! Within 3 prompts it had built a little in-browser example so I could test the functions, and then I could copy them over to my component.

![Desktop View](/assets/img/StreamYard/Notes/ClaudeAiTextEditor.png){: .normal}



## Adding the Notes Tab
