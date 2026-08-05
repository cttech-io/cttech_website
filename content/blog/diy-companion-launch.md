---
title: "DIY Companion is here: five essential DIY tools, one tidy app"
date: 2026-08-05T09:00:00Z
draft: false
categories: ["Tech"]
tags: ["iOS", "Indie Dev", "App Store", "Swift"]
description: "The app I've been building is live: a spirit level, wall scanner, torch, noise meter and ruler in one clean, ad-free app. Here's the story, and a launch offer."
---

Every so often a job around the house needs a tool you don't have to hand. Is that shelf level? Where's the stud behind this plasterboard? What *is* that at the back of the cupboard? Your phone can already answer all of this. It just needs the right app.

That's **DIY Companion**: a spirit level, a metal and wall scanner, an inspection torch, a noise meter and a ruler, in one clean app. No sign-in, no clutter, no nonsense.

<div class="app-shots">
    <img src="/images/diy-companion/01.webp" alt="Spirit level: hang shelves and frames perfectly level" width="230" loading="lazy">
    <img src="/images/diy-companion/02.webp" alt="Wall scanner: find studs, metal and wiring before you drill" width="230" loading="lazy">
    <img src="/images/diy-companion/04.webp" alt="Noise meter: check how loud it really is" width="230" loading="lazy">
    <img src="/images/diy-companion/06.webp" alt="No ads, no tracking, no subscriptions" width="230" loading="lazy">
</div>

## Why I built it

This started when I moved into my first house. I had a box of hand-me-down bits and not much else — none of the proper kit you slowly accumulate from trips to the hardware shop, and every job seemed to need something I didn't own yet. Hanging a shelf meant guessing whether it was level. Putting up a TV bracket meant drilling and hoping.

So I did the obvious thing and went looking for apps. What I found was pretty dispiriting. Half of them wanted a monthly subscription for a spirit level — a spirit level, charged by the month, forever. The other half were free but so buried in adverts that you'd tap a full-screen video before you could read a measurement, and then again when you switched tools.

None of it felt proportionate to the job. These are small, simple tools that your phone's sensors can already do perfectly well. I wanted the version that just opens and works, that I pay for once and then never think about again. It didn't seem to exist, so I built it.

## What's inside

**A precision spirit level.** Level shelves, frames and TVs with confidence. It uses motion smoothing to filter out hand-shake, so the reading holds steady, and it buzzes the moment you hit true level or plumb, with no need to stare at the screen.

**A metal and wall scanner.** Using the iPhone's built-in magnetometer, it locates studs, screws, nails, wiring and pipes behind the wall, so you drill in the right spot the first time, not the third.

**A bright inspection torch.** A steady, full-brightness light for behind furniture, under the sink, or into the dark corners the torch shortcut never quite reaches.

**A noise meter.** Measure sound levels in decibels. Handy for checking how loud a room, a power tool or a noisy appliance really is.

**A ruler.** A quick on-screen ruler, calibrated to your iPhone, for those measurements you need when there isn't a tape measure in reach.

## No ads. No subscriptions. No tracking.

Here's the part I care about most. DIY Companion is a **one-time purchase**. You pay once and own it: no subscription, no adverts, no accounts, and nothing about you is collected or tracked. It works completely offline.

That's a deliberate choice. A utility you paid for shouldn't spend its life nagging you or harvesting your data. It's also why the whole thing is about a megabyte and needs no network permission at all.

If you'd rather read the code than take my word for it, [the source is on GitHub](https://github.com/cttech-io/DIYCompanion).

## Written entirely in Swift

The whole app is Swift and SwiftUI, with no third-party dependencies whatsoever. That's not purism for its own sake — it's the reason for most of what I like about how it turned out.

Being native means talking to the hardware directly. The spirit level reads the accelerometer through Core Motion, the wall scanner reads the magnetometer, and the noise meter uses the system audio stack. There's no bridge or runtime in between, so readings arrive with no perceptible lag and the haptics fire at the exact moment you hit level rather than a beat later. On a tool you're holding against a wall, that difference is the whole experience.

It also keeps the thing genuinely small. The download is about a megabyte, which is roughly the size of one photo. Apps like this routinely run into tens or hundreds of megabytes, and most of that weight is frameworks and SDKs rather than app.

And it's the honest answer to the privacy question. There are no analytics SDKs in DIY Companion because there are no SDKs at all — nothing was ever added that could phone home, so there's nothing to disable or opt out of. Fewer moving parts also means fewer things to break when iOS updates, which matters when it's one person maintaining it.

## Launch offer

DIY Companion is normally £3.99, but it's **£1.99 until 6 September** as a launch offer. This is my first app and it has no reviews yet, so the early price is there to make it an easy decision. If it looks useful, now's the moment.

<p class="app-badge"><a href="https://apps.apple.com/gb/app/diy-companion/id6771707298"><img src="/images/appstore-badge.svg" alt="Download on the App Store" width="180" height="60"></a></p>

Requires an iPhone running iOS 17 or later.

If you give it a try, I'd genuinely love your feedback, and a rating on the App Store helps a small indie app more than you'd think. Happy building.

*Sensor accuracy depends on your device's hardware and calibration. DIY Companion is not a substitute for professional tools on safety-critical work.*

---

<small>App Store is a registered trademark of Apple Inc. iPhone is a trademark of Apple Inc., registered in the U.S. and other countries.</small>
