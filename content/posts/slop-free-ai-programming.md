---
title: "Slop-free AI programming"
date: 2026-10-27T11:23:00+01:00
draft: true
---

I've tried AI-assisted programming tools such as GitHub Copilot and Claude every now and then. I know a lot of people are very anti-AI, but I'd rather try myself than blindly trust others. In the past, whenever I tried these tools, I found them to not be very good. They made obvious mistakes and wrote code of poor quality. I like programming in Odin. In 2025 these AI tools made basic mistakes such as mixing up Odin with Go. However, when I tried Claude in August 2026, it seemed like things had gotten far enough that I could use it for real work. Claude could very quickly write correct Odin code on first try. It could also deeply analyze code and find bugs.

However, as I tried using Claude to implement some missing features in my 2D game creation library [Karl2D](https://karl2d.com), I noticed something: I slowly but surely started to accept code that Claude wrote, without properly understanding it. Something about the code Claude wrote was overwhelming me. It was somehow leading me down on a slippery slope towards _slop_.
> By "slop" I mean code that solves a problem without proper care for code quality. You don't have involve an AI to generate slop. It just means that you didn't pay enough attention to what the code actually did.

Since my little slop-mishap, I have gone through all the AI-written code that made it into Karl2D and made sure it's all in good shape.

I don't want to end up in that same situation again. I could completely abandon AI programming tools. But they _are_ useful, one just has to put care into _how_ to use them. What I've done is come up with some fairly strict rules for Claude. Let's have a look at those rules, break each down and discuss it. After discussing the rules, I'll also discuss some general points surrounding AI programming.

## Rules for Claude

Most AI tools can read a file such as `AGENTS.md` or `CLAUDE.md` that contains general instructions for the AI.

The rules that I present will go into such a file. The file can be either in your project, or global for your user.

> I use a global config for Claude. I put this file in `C:\Users\Your_Name\.claude\CLAUDE.md` (Windows) or `~/.claude/CLAUDE.md` (Linux/Mac). This means that all Claude sessions will use that config.

The content of my `CLAUDE.md` is as follows:

```md
1. Never write any comments in the code. The human will review the code and write any comments themselves. If you change any code that invalidates an existing comment, then prefix the comment with `// TODO-UPDATE-COMMENT: Reason\n// ---`.
2. Always write imperative, procedural code.
3. Never commit and push unless explicitly told to do so.
```
> I also have a fourth rule that says "Always reply very shortly". But that rule has nothing to do with the outputted code, it's just that I think Claude blabbers too much.

### Rule 1: No comments

> Never write any comments in the code. The human will review the code and write any comments themselves. If you change any code that invalidates an existing comment, then prefix the comment with `// TODO-UPDATE-COMMENT: Reason\n// ---`.

This is the most important rule. And perhaps the most controversial one. The code that Claude outputs for me never has a single comment. The only time it writes comments is when it changes some code that causes an already existing comment to become outdated. In that case it will write `// TODO-UPDATE-COMMENT: blabla` in front of the outdated comment. It's an easy thing to search for in the diff.

How did I arrive at this rule?

As I said, when I tried out Claude in August 2026 the output of it became overwhelming to the point where I started accepting changes without understanding them. I was on a high-speed train to slop.

So I tried to think: What was actually overwhelming about it? Whenever I tried to review what Claude had written I noticed that I was reading both the code and the comments it wrote. The problem is: It is much better at writing code than comments. The comments are always long, verbose and tend to focus on the wrong thing. Claude does not know what to leave out in the comments. Before you even get to the code you may be fatigued from understanding the many twists and turns of its comments.

The natural next idea was to write rules in `CLAUDE.md` on how the AI should write comments. This did not work. No matter what precise instructions I gave it, it still created hard-to-read comments that emphasized the wrong things. The text itself was often hard to read. It made weird grammatical choices. The subjects and objects of sentences were often chosen in a very confusing way.
> Being a technical writer, I understand that I may be extra hard on bad comments. But really, Claude's comments are that bad. I often have to go and look at what the code does because I cannot understand the comment, after which I just replace the multi-line comment with a single line that leaves out the non-important stuff and stresses the stuff that people need to know.
> 
> It should be noted that I never let AI write any text for me, neither for my blog posts nor for [my book](https://odinbook.com). I let it check the texts for grammar and factual errors. But I don't let it directly make any of the proposed changes. I look through the list of issues that it produces and make any adjustments myself.

After a while, I tried the no-comments rule. I quickly discovered many advantages of this.

First off, I was forced to read the code more deeply, and add my own comments based on my own understanding. By removing the comments the code Claude outputted became easier to scan. I could quickly do a first pass and spot bad code and tell Claude about it. This is the most crucial moment in getting to any kind of good result: After the first draft of the code is finished, you need to quickly find anything that rubs you the wrong way and either fix it yourself or have the AI do it. If the code had those massive multi-line Claude comments everywhere, then I would not be able to quickly scan it and the whole review process would start at the wrong end.

I don't write comments everywhere Claude would do it. I would say that Claude "litters" your code with comments. In my library Karl2D, the public API procedures need good documentation, which is all hand-written by me. In the internal parts I only add comments when I feel like more context is needed. Claude would just go bananas and add multiple lines of comments on almost every internal procedure. It made the Pull Requests very hard to review, which is why they sometimes, regrettably, got merged with the incorrect mindset "it's good enough".

One criticism I've gotten regarding my no-comments rule is that "The AI uses the comments later, to give itself context". I do not find this to be true. It can just as well read the code and figure it out. I think this idea is a form of anthropomorphization: "Humans need comments, so I guess the AI does too". But the AI can quickly read all the code of the whole program and figure out why a non-obvious thing works the way it does. It hardly needs any comments. Yes, there are rare cases where it is impossible to understand why something works the way it does. In those few cases you can add the comment yourself. However, those few cases are not good justifications for littering your whole code base and making reviewing hard.

Also, the comments Claude writes tend to cause a chain reaction. As I said, it often does not know what to include or leave out in comments. So the comments become very verbose. The comments it writes may contain details that are _not important_. But when it later sees those details spelled out in a comment, it will start thinking that they are important and try to preserve completely unimportant aspects of the code. It tricks itself! So these comments are bad _for everyone_. They only serve to give programmers who hate writing comments a false sense of accomplishment: "hah, the comments are written, this is so easy!!".
> Let's be a bit conspiratorial for a moment here. It can easily feel like the comments are there to overwhelm the programmer in order to make them accept anything and go total slopvibecode. This would be a good thing for the AI companies as they then make everyone completely dependent on the AI tools. The programmers no longer have any idea what the code does (I guess they aren't programmers anymore...). They _must_ have the tools. I don't believe this conspiracy theory to be true. But in my case it's certainly how things would work: If I didn't forbid all comments from Claude, then I would have to either stop using it or go full vibecode, because I do not have the mental capacity to review the code and the poorly written comments.

Another criticism of the no-comment rule is that the comments are needed in order for you to understand what Claude did. Given that you are working on a constrained enough feature, this is false. You told the AI what to do, so you will have a good idea of what the code is supposed to do. If you cannot understand what the code does, then perhaps the feature is too big and needs to be broken down into steps. Also, you can ask the AI about implementation details you don't understand. When asking the AI, you'll often realize that the AI has assumed some usecase that doesn't actually exist. When you point this out to the AI, the code can often be simplified a lot, to the point where you completely understand it.
> It may also be that a programmer is just too inexperienced to understand what the code does, even when simplified. I'll discuss this towards the end of the article.

### Rule 2: Always write imperative, procedural code

This rule is here because I tend to write code using the Odin programming language. A lot of the training data for AIs steers them towards more functional or object-oriented styles. So this rule is there to nudge the AI towards a mindset that is more Odin-friendly.

### Rule 3: Never commit and push unless explicitly told to do so

I always commit myself and write the commit message myself. Early on I let Claude commit. Just like its comments, it loves to write really big commit messages that are unreadable and that emphasize the wrong things. This is another form of slop: AI commit message slop.

I review the changes locally, before committing. This also helps avoid slop. By reviewing things as separate commits, you're less likely to get overwhelmed by a giant PR at the end.
> Make sure you have a good tool viewing uncommitted changes. It can be a separate diff tool, or a feature in your text editor.

## What architecture?

Whether or not people get good results out of AI programming often comes down to the code base in which they try to use it.

If you have an existing code base with some obvious architecture, then the AI will have an easy time slotting in new features. For example, say that you have an abstraction for platform support. You already support Windows and Linux. The AI will have quite an easy time adding for example Mac support, since the architecture is already there. It won't do everything perfectly, but with the no-comment rule you'll be able to quickly scan what it did and give it feedback.

On the other hand, say that you do not have an obvious architecture. Perhaps you're starting a completely new project. In that case you have to be much more careful. I'd recommend asking the AI to not do any fancy architecture at all. But you must still review its code and make sure it seems reasonable. When you feel like architecture is needed, then you can type out the suggested API surfaces and let the AI restructure things according to it. An important insight here is that the AI has encyclopedic knowledge, but no taste. You have to bring the taste and the architecture. So in new projects, you need to be much more hands-on. Perhaps you should even write most of the basics completely by hand before involving any kind of AI help. Try to find some balance where you don't feel like you lose control.

## How much faster?

AI programming can be a lot faster than what humans can do. But the speed depends a lot on the type of task and the existing architecture.

For non-trivial bugs, I'd say that the AI is 30-100 times faster than me. Claude has often found the root cause for a bug within a minute, when it would have taken me 60 minutes to catch the bug. This assumes it is a non-trivial bug that takes much more work than just catching it in a debugger. Claude has really replaced debuggers for me. While I love RAD Debugger and think it is a good tool, I haven't actually used it for several months.

For benchmarking, Claude also beats me many times over. Setting up tests, measuring many different scenarios and summarizing the findings is done better than I'd ever do it myself, and faster.
> An annoying thing when Claude is testing and benchmarking my Karl2D library is that it tends to open and close windows all the time. It is creating lots of little test programs that need to be properly started. However, I'm supposed to be able to do other work while Claude is doing things. I can't run those tasks in "the cloud" because my library needs a real GPU etc. A solution to those kinds of problems is to have a separate computer for such AI tasks. You can then remote control it, for example using the Remote Control feature of Claude Code. Perhaps it would be possible to use a virtual machine that has GPU access? On Linux you may be able have Claude run the test applications within an offscreen compositor.

When it comes to writing new code, the speedup versus doing it "by hand" is smaller. I don't have any sure estimates for how much faster the AI will be. However, let's assume that the AI is 20 times faster than a human _if_ you go full vibecode and don't care about the code. But since vibecoded programs are of poor quality, the development speed will decrease over time as complicated issues surface.
> Vibecode means that you don't look at the code at all. You only judge the software by how it works while running, and then you tell the AI what you want to change. Over time, it leads to very poor code that may crumble under its own issues. You can always have the AI completely rewrite it when it becomes too much of an issue. But what if you had sold it to customers and the rewrite doesn't work the way the old customers want? You've totally lost control!

Assuming we use my three AI programming rules and make sure we understand any generated code, how fast will the process then be? Again, I don't really know. But around 5 times compared to doing it manually does feel like a good lower estimate. That's still a _really good number_. You don't want to be as fast as possible, because you have to sacrifice some of that speed for quality.

## What AI models?

I have only tried Claude recently. I mainly use the Claude Opus 5.5 model. I can't really say how other models perform.

One thing I _can_ say is that the free AI models will not result in any good work. You can use such models to ask basic questions. But when you try to solve complex problems, the free models will end up "running in circles", restating the same solution you've already ruled out. If you want to do any kind of deeper work or get quality code written, then you unfortunately have to use the paid models.

## The junior-senior paradox

As AI is getting more popular, there is a grim outlook on the job market. There is an idea that programming in the future will largely be senior programmers that control AI agents and review their work. I believe that the rules I spelled out in this article make that job of "overseeing" the AI easier, making the likelihood of slop smaller.

This brings us to a strange problem: With AI programming, junior programmers don't have to understand code as deeply as before. They can get stuff done anyways. Assuming that we still need seniors that oversee AI in order to produce quality code, how do juniors get there?

I don't know for sure. I got my programming experience from years of grinding and making sure I understand problems as deeply as my mental capacity allows. If I skipped that part, then I would not be able to quickly scan Claude's output and give it pointed feedback.

Some seniors would say something like "juniors should not use AI tools at all, those tools are for seniors... Something something with great power comes great responsibility". You are free to think like that, but it's not how things will work out. To juniors that just seems like gatekeeping technology, whether or not that is true.

My suggestion to any junior programmer is as follows:

- It's OK to use AI tools. Try the ideas in this article, see if it makes it easier for you to review and understand the code. Ask the AI about the code it wrote, figure out why it did things. Learn from understanding the code. If the code is very confusing, then challenge the AI about why it works that way, sometimes it does incorrect assumptions that leads to overly complicated code.
- Have some AI-free side projects. Try to build some smaller projects completely without any AI-assistance.

The idea is not to go hardcore in either direction. Use AI tools in such a way that you learn from the resulting code. On top of that, in order to train your independent thinking, have some AI-free projects as well.