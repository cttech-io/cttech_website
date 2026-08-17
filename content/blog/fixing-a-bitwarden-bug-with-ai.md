---
title: Using AI to Find and Fix a Real Bug in Bitwarden
date: 2026-08-17T10:00:00Z
draft: false
categories:
  - Tech
  - Open Source
tags:
  - Bitwarden
  - AI
  - Open-Source
  - Self-hosting
  - Claude
description: A routine self-hosted update crashed, so I used AI to trace the fault into Bitwarden's own source, find a threading bug, and open a fix upstream.
---

I run my own self-hosted [Bitwarden](https://bitwarden.com/) server. It holds every password I have, so I keep it patched and don't take chances with it. A couple of weeks ago, gone 9pm, about to call it a day, a routine update went sideways: a crash, a couple of hours hunched over a stack trace in someone else's source code, and eventually a pull request against Bitwarden's official repository fixing a genuine bug.

The twist is I did the whole thing with an AI assistant, [Claude Code](https://www.anthropic.com/claude-code), doing the driving. I don't mean "AI wrote my blog post." I mean AI reproducing a fault, reading a stack trace, finding the bug in Bitwarden's code, and helping me ship a fix upstream. 

## The update that wouldn't

Updating self-hosted Bitwarden is normally two commands:

```
./bitwarden.sh updateself
./bitwarden.sh update
```

This time the second command got about halfway, stopped all my containers, started pulling the new version, and then died with this:

```
Unhandled exception. System.ArgumentException: Destination is too short. (Parameter 'destination')
   at System.Text.StringBuilder.AppendWithExpansion(...)
   at System.Text.StringBuilder.AppendLine(String value)
   at Bit.Setup.Helpers.<Exec>b__0(Object _, DataReceivedEventArgs e) in /source/util/Setup/Helpers.cs:line 132
```

The important part: this happened after it had shut down and removed every container, but before it touched any of my data. So my vault was down, but nothing was corrupted. My first reaction was a fairly un-technical string of swearing. Not how I'd planned to spend the evening, but at least nothing was actually on fire. First priority was getting it back online, which turned out to be one command; Bitwarden had kept the old version's config sitting there, and my password manager was back within seconds. Small mercies.

Now I had a working vault on the old version and a very specific error message. Time to work out what had gone wrong.

## Reading someone else's code

This is where having an AI assistant helped most. I'd never worked with, let alone written, a line of C# in my life, and that stack trace pointed at an exact file and line in Bitwarden's source: `util/Setup/Helpers.cs`, line 132. We pulled up that file straight from Bitwarden's public GitHub, and the culprit was a small helper that runs a command and captures its output:

```csharp
var result = new StringBuilder();

process.OutputDataReceived += (_, e) => { ...; result.AppendLine(e.Data); }; // handles normal output
process.ErrorDataReceived  += (_, e) => { ...; result.AppendLine(e.Data); }; // handles error output
```

The useful part wasn't the code itself, it was getting there. I can read a stack trace well enough to know it's pointing at a specific file and line, but I have no intuition for .NET's event model, and no idea what `OutputDataReceived` or `ErrorDataReceived` even are. Claude pulled the actual file from Bitwarden's GitHub rather than guessing at what might be there, matched it against the stack trace line by line, and translated what it found into terms I could reason about: two events, fired on separate threads, both writing into the same object. That's the part that would have taken me the better part of an evening on my own, assuming I didn't just give up and file a vague bug report instead.

In plain English: when a program runs another command, that command can talk back on two channels, normal output and error output. This code listens to both and writes everything into one shared notepad (`StringBuilder`).

The problem is those two channels are handled on two separate threads, two things happening at literally the same time, and that "notepad" isn't designed to be written to by two threads at once. Most of the time you get away with it. But if the command spits out normal text and an error at the exact same instant, both threads scribble on the notepad simultaneously, the notepad's internal bookkeeping gets corrupted, and the whole process falls over. That's the `Destination is too short` crash. In programming terms it's a classic race condition. (I needed it explained twice before it clicked.)

## The bug proves itself

I was fairly confident of the diagnosis, but I wanted real confirmation before telling strangers on the internet their code was broken. So I retried the update, more out of curiosity than any real confidence it would help. It crashed again, in exactly the same way, except this time the error pointed at line 138 instead of line 132. I'd have skimmed straight past that on my own, half paying attention to a terminal at 10pm. It took asking whether the two crashes actually lined up to realise they didn't, and that not lining up was the useful part.

That was the smoking gun. Line 132 is the *normal output* handler; line 138 is the *error output* handler. Crashing in both, on different runs, is exactly what you'd expect if two threads are fighting over the same notepad: whichever one happens to be mid-write when things go wrong is the one that dies. It also told me something important about the fix. Protecting one handler isn't enough. You have to protect both.

(As a bonus, this explained why the update failed so reliably for me when plenty of other people update fine: this particular version's setup step produces a lot of simultaneous output, so a collision that's normally rare became almost guaranteed.)

## The fix

The fix for "two threads mustn't touch this at the same time" is to make them take turns. In C# that's a `lock`: a way of saying only one thread in here at a time.

```csharp
var resultLock = new object();

process.OutputDataReceived += (_, e) => { ...; lock (resultLock) { result.AppendLine(e.Data); } };
process.ErrorDataReceived  += (_, e) => { ...; lock (resultLock) { result.AppendLine(e.Data); } };
```

Four lines. Both handlers share one lock now, so they can't scribble over each other. It doesn't change what the code does in the normal case, it just stops the two threads colliding. Small fix, in hindsight, though it took a while to get there.

From there it was standard open-source contribution mechanics: fork Bitwarden's repository, make the change on a branch, write a clear description with the stack traces and the root cause, and open a pull request. Drafting that description was the same story as the diagnosis: not writing it wholesale, but pulling both stack traces back out, laying out the root cause in a way a maintainer skimming fifty PRs a day could follow quickly, and getting the tone right for a stranger's repository I'd never contributed to before. I read it over, cut a couple of lines that sounded like I was trying too hard, and sent it. That's [bitwarden/server #8141](https://github.com/bitwarden/server/pull/8141), merged into `main` six days later, approved by two Bitwarden engineers.

## Where the human still mattered

This is the part I actually want to get across, because "AI fixed a bug" is the boring headline, and not quite true anyway. The more interesting bit is which decisions stayed with me.

- Whether to keep chasing the update at all. Because this is a race, I could have kept retrying until I got lucky, and for about ten minutes that's exactly what I was doing. I stopped myself: every failed attempt is a brief outage on my password vault, and the new version had nothing security-critical in it. Holding on the old, working version was the sensible call, once I actually stopped and thought about it instead of just reacting. An AI won't make that trade-off for you; it's a judgement about your risk.
- Whether to trust a home-built patched version. One option was to build my own patched copy of Bitwarden's updater and run that instead. Technically fine, but do I want a locally built binary sitting in the update path of the thing that guards all my passwords? For a low-value update, no. That's a trust decision, not a technical one, and it's the kind of thing that's easy to talk yourself into at midnight if you're not paying attention.
- Signing the contributor agreement. Contributing to Bitwarden means signing their [Contributor License Agreement](https://cla-assistant.io/bitwarden/server), under my own name and my own legal responsibility. No assistant can or should do that for me.

So the division of labour looked something like this: the AI did the fast, mechanical, error-prone work, reproducing the fault, reading a stack trace, navigating an unfamiliar codebase to the exact line, spotting a subtle threading bug, drafting the fix and the PR. I did the judgement calls: what's worth doing, what's safe to trust, what I'm willing to put my name on. It's a tidier split in hindsight than it felt like at 11pm on a Tuesday.

## The takeaway

A year or two ago, "my password manager's updater is crashing" would have ended with me either waiting for someone else to notice, or spending an evening spelunking through an unfamiliar C# codebase I've never worked in, probably giving up around midnight and just reinstalling from scratch. Instead it ended with an actual fix, now merged, that helps every self-hoster who'd have hit the same wall.

I don't have some big unified theory about what this means for AI and software. What I know is that a bug I'd normally have shrugged off, retried a few times, and quietly worked around, turned into something I actually fixed. That's a small thing, but it's not nothing.

If you self-host anything and hit a bug like this, it's worth at least checking whether the fix is somewhere you can actually get to, in code you can read, in a repository you're allowed to change. Doesn't always work out that way. This time it did.
