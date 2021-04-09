---
layout: post
title:  "Getting Started With Frame"
date:   2021-03-15 00:00:00 -0800
categories: language-basics
---

## What You Need to Know

Documents created using Frame and the Framepiler are called _specs_ for *specifications*. This is because Frame is not (currently) a complete programming language - it's focus is on creating structure and crafting how a system should respond to events, not data manipulation. To that end, Frame enables developers to generate a *Spec* base class (currently for 6 languages - more to come) that has all of the state machine logic but not all of the final action implementation code needed for a functional system.

The spec class contains stubbed out _action_ methods that need to be implemented by developers to be fully operational. This is easy to complete as the full controller can just inherit from the spec base class and then have the missing action functionality added:

<img src="https://cdn.jsdelivr.net/gh/frame-lang/article_content@18c507458d67cb1c486752ac565805fb426005e6/frame-lang.org/assets/img/controller_inheritance.png" />

In fact, the Framepiler generates a stub of this derived controller class for you (look at the bottom of the generated spec files). There you will find the action methods that just need to be filled out. This approach makes it simple to get started developing Frame controllers.  

## Three steps to Create a Frame System Controller

The steps to create a working Frame system controller are straightforward:

1. Download the supporting class file for your target language from [GitHub](https://github.com/frame-lang/frame-ancillary-files).
2. Create a Frame spec using the online [Framepiler](http://framepiler.frame-lang.org) and generate the target language spec controller.
3. At the bottom of the generated file is a commented out stub class that derives from the base spec class. Uncomment it and add code to the actions.

## Watch!

I've put together videos on YouTube to show step-by-step how to get started with Frame.

## Start Small

 When starting to learn Frame, _start small_! You are going to be retraining yourself to think about programming in completely different way, so it is easiest to take it in bite sized chunks. Instead of attempting to reimplement your most complex hairball of a class first (which is probably where you really want it the most), find a short workflow or simple UI controller to begin with.

 Much of the challenge arises because state machines compartmentalize a class in a way that is very different than the typical class implementation. Learning to think about compartments (states) and how to craft a system using them is a very different (but hugely rewarding!) way of creating software.

## Follow Patterns

 The Frame ecosystem and community will be rapidly expanding the number of freely available "solutions". Most likely your problem is not dissimilar from something someone else has created.

## Join the Community

 I look forward to helping and collaborating with others who are curious, facinated, doubtful or passionate about creating stateful software - using Frame or not.

Come join the discussion on [Discord](https://discord.com/invite/CfbU4QCbSD) or reach out to me on [Reddit](https://www.reddit.com/r/statemachines/) and let me know what you are interested in doing, wondering about or need help with using Frame.



You can find more at my [Art of the State]
## **Happy Framing!**
