---
title: "CursorAI and Other Tempting Alternatives"
date: 2025-02-11 08:42:00 -500
categories: [ Bending Spoons, TypeScript]
tags: [production, RxJS, Redux, my journey]
image: "https://raw.githubusercontent.com/cristinaponcela/cristinaponcela.github.io/refs/heads/main/assets/img/ides/windsurfVScursor.jpg"
---

# What's the best AI integration for an IDE?

So as promised when I wrote about Xcode Completion and Copilot for Xcode, I have finally given CursorAI a shot. And by a shot I mean I have _used_ it. Like, every day. For 2 months. It has saved my ass countless times already.


### Cursor's Main Differentiating Advantage

Get ready for this. It's crazy. CursorAI is integrated in your IDE like Copilot would be in VSC, or Cascade in Windsurf (run cmd + L). But the main feature that makes Cursor ply in a completely different league is it can access your code, and it is extremely accurate when doing so. By including things like @Codebase, the filename or the function or variable name in your prompt, Cursor immediately identifies which files are relevant, presents them to you, and forms a conclusion to reply to your query. 

Now, this by itself is incredible. I no longer have to copy paste code into ChatGPT or Claude? Amazing! But my particular favourite use of this is using the Cursor chat as a search function.

![Desktop View](/assets/img/ides/Cursor-Chat.png){: .normal}

See this example: in a codebase with 14 million lines of code, I ask Cursor to find a function by describing its functionality. It does so in 2 seconds!!!

<iframe class="embed-video" loading="lazy" src="/assets/img/ides/search-codebase.mp4" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen=""></iframe>


Also, Cursor integrates, as per usual, with your favourite model of choice. With the free plan, you get limited queries in ChatGPT's o1 mini model, with quote unquote improved reasoning. But honestly, paying for Cursor pays for itself probably in the first few hours of use. It increases efficiency so much I highly recommend you get the paid version. My preferred integrated model is Anthropic's Sonnet-5, but it works well with any.

Another great characteristic of Cursor: it has fantastic inline completion. Be it for blogs, docs or code, it tends to be very accurate, with elegant and succint code recommendations. 





After ranting about all the advantages CursorAI offers, and why - spoiler alert - it is my favourite AI integration by far for any IDE, I want to give a few other alternatives.

Apart from Xcode Completion and Copilot for Xcode, I have reaaaally enjoyed Windsurf. 

Windsurf is an AI-powered IDE develpped by Codeium. As per usual, Windsurf integrates AI models like GPT 4o and Claude 3.5 Sonnet to provide its core AI capabilities. However, Windsurf has also developed its own models, including Cascade Base and Codeium Fast.

![Desktop View](/assets/img/ides/windsurf-chat.png){: .normal}

It runs off of VSC, just like CursorAI, which makes integration seemless as you can import your VSC extensions to use your set-up of choice from the get-go, as well as your settings. This is super helpful when you have things like Prettier Format on Save, or Jest Runner, etc. 

Also as Cursor, they come with bougee themes (background colours) for your IDE. Not a massive plus, but keeps things interesting. My personal favourite is Midnight Sky (Solarized Dark in Cursor).

But perhaps the best feature of Windsurf, it has _really_ good inline completion, like Cursor, but one an autocompletion is added (pressing tab), it then produces another autocompletion based on the new input, and so on. Super useful when writing a blog like this, since it will give me full code blocks as examples to the concept I'm trying to explain by pressing tab a few times.

<iframe class="embed-video" loading="lazy" src="/assets/img/ides/windsurf-multiple-autocompletions.mp4" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen=""></iframe>

Windsurf is quite good at understanding context, and will take previous code in your file to predict the next autocompletion. For example, and because you're going to have a blast seeing the behind the scenes of how these articles are written, look at how it autogenerates lines for this blog:

<iframe class="embed-video" loading="lazy" src="/assets/img/ides/windsurf-autocompletion.mp4" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen=""></iframe>

Again, same as Cursor, apart from inline completion we have an integrated AI chat. By just pressing `Command + L`, we can open it and start prompting our model of choice for help. It makes using AI for work a lot more efficient than copy pasting into the AI's browser chat.

<iframe class="embed-video" loading="lazy" src="/assets/img/ides/windsurf-chat-video.mp4" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen=""></iframe>



## Why Cursor wins the AI IDE war

The jewel of the crown. This feature single handedly has saved me countless hours of work alread, plus a few headaches and a lot of frustration. Cursor's chat allows you to give it specific folders, files or the codebase as a whole as input! Meaning your prompts are answered waaaaaaay more accurately, and you can also use it for searching the codebase. This is my single, most favorite lifehack. And considering I am currently working on a product with over 14 million lines of code, I can assure you it has not only made my work efficient, but possible.

<iframe class="embed-video" loading="lazy" src="/assets/img/ides/search-codebase.mp4" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen=""></iframe>

As you can see, by adding `@` and whatever part of the codebase you want Cursor to access, it takes only a few seconds to generate code recommendations or find the requested code. AND the response is clickable, to save you further time!


## An Honorable Mention to XCode

There are 2 main features I wish would be implemented into Cursor, both present in XCode, that would make it my indisputable favorite.

First, XCode has a very efficient way to allow you to find compilation erros in your code. The second you run `Cmd + R` to reload your Simulator, it will show you all of them, in the same place, within a second.

<iframe class="embed-video" loading="lazy" src="/assets/img/ides/XCode-errors.mp4" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen=""></iframe>

This makes debugging very straightforward, and saves a lot of time in fins=ding all the errors and their root cause. This is especially easy to do with Swift, as no imports are needed, so the interrelations between files are made easier through tracing errors here.


Second, XCode has a pretty cool side bar where you can see errors marked in red, and autogenerated "chapters", as if your code was a Google Docs. Again, this makes it easy to find errors, and makes the codebase a lot more comprehensible.

![Desktop View](/assets/img/ides/XCode-Chapters.png){: .normal}