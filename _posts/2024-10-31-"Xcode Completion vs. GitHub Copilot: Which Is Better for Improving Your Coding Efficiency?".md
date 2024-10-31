---
title: "Xcode Completion vs. GitHub Copilot: Which Is Better for Improving Your Coding Efficiency?"
date: 2024-10-31 08:42:00 -500
categories: [Swift, Xcode, GitHub Copilot, AI Tools]
tags: [comparison, for beginners, recommendations, efficiency]
image: "https://raw.githubusercontent.com/cristinaponcela/cristinaponcela.github.io/refs/heads/main/assets/img/CopilotForXcode.png"
---

The deeper I dive into the software industry, the more I realize that having the right tools is just as important as being smart. Sure, you can be a computer wiz and build amazing apps, but you can get just as far if you know the right packages, the most efficient application toolkits and, most importantly, the best AI tools. And what’s more - you may even arrive at the same destination faster; in the world of coding, saving time without sacrificing quality is a huge win. 

This post should show a thorough (and biased) comparison of the 2 tools that have been using a lot recently in Xcode: Completion and GitHub Copilot. 

I mainly use Copilot for Xcode hooked up to ChatGPT using an OpenAI API key. This allows me to both enjoy inline code autocompletion and prompt generative responses to directly insert into my code. Lovely stuff.


## What is Xcode Completion?

For the newly initiated, Xcode is Apple's native IDE, and it comes with its own AI-powered completion extension, Xcode Completion. 

It focuses on Swift and Objective-C, so it provides a very tailored experience; it used to be the coolest thing for Apple Developers, as it was the only AI tool that was natively integrated with Xcode when the GPT craze broke out and every startup, with a .ai domain, was claiming they used Machine Learning. 

I have to give it to Apple, they are usually extremely annoying with how buggy their native extensions are, but Completion does work pretty well. However, it feels a bit outdated, more like a glorified Emmet expansion than a powerful AI tool in its own right.


## What is GitHub Copilot?

GitHub Copilot is an AI-powered coding assistant developed by GitHub and OpenAI. It helps write code faster by generating real-time code suggestions, using machine learning trained on vast amounts of public code to provide web-informed assistance. 

Other options like adding a Chat Model, Custom Commands or Key Bindings help to further blow up your efficiency. 

But until recently-ish, it couldn’t compete with Xcode Completion as a tool for Apple Developers for the simple reason that it wasn’t available.

Insert the absolute god-sent that is the GitHub handle @intitni, who last year created the Xcode Source Editor Extension [“Copilot for Xcode”](https://github.com/intitni/CopilotForXcode). This completely changes the game. I have bought them a coffee, and [you should too](http://buymeacoffee.com/intitni). My productivity since has skyrocketed, as has how enjoyable coding has become. It is way more streamlined than having to go back and forth from a GPT online, and way more powerful than simple inline autocomplete.

[Now alongside Swift,](https://github.blog/changelog/2024-10-29-github-copilot-code-completion-in-xcode-is-now-available-in-public-preview/) it provides code completion suggestions across multiple languages and frameworks, including Python, JavaScript, TypeScript, Ruby, Go, C#, and C++.


## 1. My Experience Using Them

I’ve very recently started to learn Swift for a project I am working on. Not gonna lie, each time I have to switch to a new language for whatever current project I’m working on is a hassle - I’m a mathematician, and as such I care about the logic, not the latest syntax that has become popular. So I don’t tend to spend much time learning the language, instead grasping the basics and using pseudo code and AIs for the rest. 

This is why Xcode Completion sucks. It is only really helpful if you already have a pretty solid understanding of Swift syntax, and for me the whole point of AIs is to streamline learning and allow you to do things that were impossible just by yourself before. So don’t get me wrong, it works well as an autocomplete, but it is literally what it says on the tin. It won’t go much further than that, and you’ll be lucky if it even gets the word right, let alone the whole line with variables and the desired functionality.

<iframe width="640vw" height="420vw" src="/assets/img/XcodeCompletion.mp4" frameborder="0"> </iframe>

As you can see here, it takes a full 3 characters to notice the line before a closing bracket of a function is return, and provides no further input. Genius unlocked.

<iframe width="640vw" height="420vw" src="/assets/img/CopilotWorks.mp4" frameborder="0"> </iframe>

Meanwhile, I have found GitHub Copilot to perform a lot better. As you can see here, I had a line and deleted it, but Copilot quickly suggests a full line with the correct variables and functionality.  The 2 lines match.

That’s not even close to being all Copilot has to offer! Every time I play around I find new cool features. For example, see below Code Snippets, that gives you examples of widely used structures upon request:

![Desktop View](/assets/img/CopilotCodeSnippets.png){: .normal}

And now onto my actual favorite feature… ChatGPT integrated in my IDE! Truly a dream come true.

<iframe width="640vw" height="420vw" src="/assets/img/CopilotChatGPT.mp4" frameborder="0"> </iframe>


You can add line or code specific prompts and it will give you the code you can easily and efficiently paste over, without needing to copy paste back and forth between your files and the ChatGPT app or web.

Do be advised though, you will need to buy OpenAI credits for this functionality, though it is super worth it. Starting on tier 1, $5 gives you 500 requests (with a maximum of 500 Requests Per Month).


## 2. A Comparison of Key Features

#### Apple Completion: A Reliable Assistant with Limited Flexibility

Apple’s completion tool is useful for fast and accurate syntax-aware suggestions, but it feels somewhat “static.” Again, to me it’s just a glorified HTML Emmet expansion that wants to ride the AI wave - it takes your typed keywords and provides standardized completions. While great for speed, this limits its use. It do be free tho!

#### GitHub Copilot: AI-Driven and Web-Powered Suggestions

Copilot shines when it comes to adaptive learning. It continuously learns from the codebase you're working in and other public code it’s been trained on, making it much more versatile. Plus, because Copilot has web access, it stays current with the latest libraries, frameworks, and code patterns - far beyond what Apple’s built-in completion can offer. But you will have to pay for both Copilot and, in case you want to use the Chat Model, OpenAI credits.


## 3. How GitHub Copilot Helps Improve Efficiency

- Quick Syntax Completion: auto-fills methods, variable names, and common structures.

- Error Minimization: helps avoid syntax errors by preemptively filling in the required elements, in a fast and responsive way.

- Perfect for Basic Development: ensures best practices, especially useful since Swift can be a pain.

- Expansive Code Suggestions: can fill out entire blocks of code and even full functions. It learns from your project and similar code.

- Adapts to Project-Specific Needs: you can prompt Copilot for specific tasks or documentation, and it will adapt to your style. Want comments added to your function? Copilot can provide those based on what it thinks the code does.

- Constantly Updated: because it learns from an online dataset, Copilot’s suggestions include recent libraries and best practices, keeping your code current.


## 4. How Beginners Can Use Copilot to Boost Efficiency

If you’re new to coding, both tools can be invaluable in helping you learn quickly. They helped me. Apple Completion will be particularly useful for fast learning and familiarization, as it is more syntax aware. But I recommend you go straight to Copilot; here’s how you can leverage it in your workflow:

* Quick Feedback Loop: every time you type, you get instant feedback - this constant feedback will help you avoid simple errors and learn quickly.

* Ask for Help with Prompts: dude, we live in a day and age where you can ask your code to code and it will code. It responds to natural language prompts! Stuck on a function? You can prompt it to “create a function that fetches data from an API” and let it suggest a solution.

* See How Experienced Developers Code: by watching the suggestions Copilot makes, you get a front-row seat to common coding patterns and idiomatic practices. This is how I started to code back in the day, I just treated ChatGPT as my lecturer. Or you can ask Copilot to add comments to everything.

* Explore Libraries and Frameworks: Copilot has a much larger scope for suggesting library functions and third-party frameworks, which can introduce you to new tools without scouring through documentation. Though reading docu is fun and will get you more involved in the community, so try not to cut it out completely.


## 5. When to Use Each Tool: A Practical Guide

Let’s look at some scenarios to see when Apple’s completion shines and when Copilot takes the lead:

#### Scenario 1: Writing Basic iOS App Code

Apple’s native completion is your best friend here. If you’re setting up standard structures in Swift or filling out boilerplate code, it’s fast and focused.

For example, type func viewDidLoad() and let Apple completion take over for standard lifecycle methods and properties.


#### Scenario 2: Implementing Complex, Unfamiliar Logic

Copilot excels in this area. Let’s say you’re creating a function to handle custom data parsing. You can start typing your code, and Copilot will often fill in suggestions based on how similar functions are typically structured.
If you want to go further, try prompting Copilot with “write a Swift function for data parsing with error handling.” This can be a lifesaver when exploring new patterns. 

It just wrote the entire Auth logic for Sign Up and Log In in around 20 minutes yesterday for my app. Though some credit also has to be given to the incredible [GRDB.swift](https://github.com/groue/GRDB.swift) here.


#### Scenario 3: Learning Through Example

As a beginner, you will benefit immensely from Copilot’s “teach-by-example” approach. As it suggests code, try to understand why it’s offering certain methods or structures. If you’re not sure, ask Copilot to add comments.

Apple’s completion, while straightforward, doesn’t have this teaching element since it’s limited to simple syntax-based suggestions.


## 6. How to Get Started

And finally, let's help you leave this article hooked up to a boost in your productivity. 

But instead of doing yet another tutorial that no one will read, let me just point you to the best resources. If you don’t have Copilot for Xcode yet, do go to the docu, as it is very rigorous and clear. ([Buy intitni a coffee!](http://buymeacoffee.com/intitni))

But if you’re a visual learner (read: “lazy”) like me and prefer a good old video tutorial, check [this one](https://www.youtube.com/watch?v=T4lWtKa9lc8) out for Copilot for Xcode and [this one](https://www.youtube.com/watch?v=4mYM702WtE0) to add the ChatGPT Chat Model. 



Also, happy Halloween! Hope you enjoyed the spooky Xcode recs.
