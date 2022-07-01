---
title: The pleasure of curating your own IDE
description: 
date: 2022-07-01
tags:
  - information
  - software
  - productivity
layout: layouts/post.njk
---

A few months ago I realized that I was not enjoying my computing experience in general. I work as a software developer and little by little, my environment turned into a  slow, overwheelming and generally unpleasant place that didn't invite creativity.

My common workflow involves opening Visual Studio Code, Jira and going through the daily tasks. I also like to have creative breaks and spend some time writing and thinking about ideas I would like to implement or just explore something cool or read about a cool essay I saw on HN and saved for later.

Interacting with dozens of UI's everyday is taxing for my brain, each one has different  UI patterns and conventions and their quirks.

Editors are no different, I like the batteries included approach that Visual Studio Code has and the generally good experience of it using default settings.

But there's a lot of stuff in the VS Code UI that I don't care for. And the idea of having something simpler started to form on my mind. I want to reduce the amount of stuff that's not directly related to my own workflow and how my brain works.

Also, I'd like to have muscular memory for the stuff I do frequently like:

- Split windows
- Hide everything that's not the current editor
- Go further and hide anything that's not the current fragment I'm editing
- Trigger snippets completion
- Smart auto complete and code actions.
- Quickly opening a terminal window (and treat it like another pane)

VSCode can do most of those (except maybe #3 hiding all the text in the current file with the exception of the fragment I'm editing but I'm sure there's a plugin for that)

At first I gave vim a try, I know how to move around and do basic stuff. The problem is that customizing things was not pleasant for me. Even if modal editing is nice, I think it ranks low on my priorities and a more elegant way to customize stuff is more important to me.

Next I gave Emacs a try and it clicked (pun non intended).

A few years ago I tried Emacs for some time but ended up declaring bankruptcy, I now realize my mistake. I tried to super charge it from the beginning and ended up with a big pile of configuration and plugins and stuff without even trying to understand the philosophy of the tool and its native features.

I think Emacs is pretty cool if you try to understand its way of doing things.

Emacs evangelists mention "discoverability" as a big advantage and I completely agree. After the initial learning curve one of the coolest things about it is that discovering new features is intuitive because of the way emacs group commands and stuff. So you keep learning while using it, that's something I didn't feel about my initial vim setup (or even VSCode) (I now wonder why **discoverability** is not something more tools strive for)

So after getting some familiarity with the basic text movement commands (I'm not using evil mode for now) I started enjoying editing text with it.


For stuff that really benefits from code completion (like typescript and react projects) I setup lsp-mode. The awesome thing about lsp is that you get the smart code completion that things like VSCode offer but without having to spend a lot of time configuring stuff, because it uses the standard language server protocol for the language, and you can install one for your preferred language.

It's true that there's some learning curve and having an initial system properly configured will take some time. I felt that during a few weeks I was kind of slow while I was getting familiar and having a printed cheatsheet was useful, eventually my brain started to learn the commands and key bindings and it felt awesome.

## It's not spectacular and that's good

The good thing about my new emacs based ide workflow is that it's very simple. Most of the time I'm only seeing the text I'm editing and that's it. I can use the `narrow` feature of emacs and "focus" only on the region I'm editing  and that's great, less stuff on the screen that's distracting me from my immediate goal. After finishing editing the region I press Control-x-n-w and I see all the file text again if I need to.

There's very few icons, colors, etc. I now consider all of that unnecesary. It can look cool but in the end it can be wasting valuable mental cycles as my eyes tend to go to colorful things even if they have nothing to do with my task.

I think emacs is "well proven boring software that works" and as I get older I have much more appreciation for things where my effort can compound over the long term (instead of re learning the same thing with different conventions again and again).



## No fancy and unwanted new features

Another benefit of curating your own IDE with boring and proven tools is that you won't have surprises when there's a new version of it.

With simple, boring and text based tools like emacs or vim you know that updates won't include anything and you can keep your environment exactly the same for decades. If you master your tool conventions, you can get compounding interest as you won't have to re learn micro operations


## It's all about removing micro distractions and making the tool work for you, not the opposite.

I think having some sense of control over the tools we use daily is important and it's good to analyze our workflow and look for tooling that enables it and is not creating resistance every few minutes. 

I'm not advocating for Emacs or vim or VSCode, I think the idea is that for critical and frequent tasks, the more aligned our tools are with the way our brain works the more mental energy we'll save for the really important stuff.


## No fancy and unwanted new features

Another benefit of curating your own IDE with boring and proven tools is that you won't have surprises when there's a new version of it.

With simple, boring and text based tools like emacs or vim you know that updates won't include anything and you can keep your environment exactly the same for decades. If you master your tool conventions, you can get compounding interest as you won't have to re learn micro operations
