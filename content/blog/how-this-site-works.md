---
title: "Moving from Site Builders to Hugo"
date: 2026-06-07T17:00:00Z
draft: false
categories: ["Tech"]
tags: ["Hugo", "DevOps", "GitHub Actions", "Web Dev"]
description: "Why I moved away from expensive site builders to a stack that's simpler and more efficient."
---

I’ve never been much of a fan of paying for things I can do myself, but it took me a while to find the right way to run this site. 

In the past, I’ve used the big-name platforms like Squarespace and Wix, thinking they’d save me a bit of time. Honestly? I found them more of a hassle than they were worth. For a relatively simple site like this, they’re surprisingly cumbersome to manage. You spend half your time fighting with a "drag-and-drop" editor that never quite puts things where you want them, and the other half wondering why you’re paying a monthly fee for the privilege. 

I couldn't justify the "subscription tax" for something I could build better myself. So, I scrapped the builders and went for a stack that’s actually mine. Here are the bits and pieces that keep this place running.

### The Engine: Hugo

I’m a DevOps engineer by day, so I’m naturally inclined to automate things until they’re almost too simple. I went with [Hugo](https://gohugo.io/). It’s a static site generator, which basically means it takes a bunch of Markdown files (like the one I'm writing now) and turns them into a website in about half a second. 

Compared to the clunky dashboards of the "all-in-one" builders, Hugo is a breath of fresh air. There’s no database to break, no WordPress plugins to update, and no server-side nonsense. It’s just plain HTML files sitting on a server. Properly simple.

### The Look: No Frameworks, No Fuss

I’m a bit of a stickler for speed, so I didn't want to load in a massive CSS framework that I only use 5% of. Since I'm more of a backend person, I used Claude to help me write the CSS—what I call "The Studio" palette. It’s dark by default, uses clean typography, and is very easy to maintain. 

The whole site is tiny. It loads instantly even if you’re on a slow mobile connection. If I want to change a color or a font, I tweak one variable and the whole site is updated. 

### The Plumbing: GitHub Actions

Since I do this for a living, I couldn’t just upload files manually like it’s 2005. I wanted a proper "set and forget" pipeline.

The code lives on GitHub. When I push a change, a GitHub Action kicks in, builds the site, minifies everything to keep it snappy, and pushes it live to GitHub Pages. It takes about 30 seconds from start to finish. 

The best part? The hosting is free. The build tools are free. The only thing I pay for is the domain name. 

### Ticking Over

The goal was to have a site that requires zero maintenance and doesn't cost a fortune. I don't want to spend my weekends fighting with a proprietary editor or worrying about whether a "Pro" subscription has expired. I just want to write, post the occasional photo, and get back to my homelab.

If you’re thinking about starting a blog, skip the expensive, cumbersome builders. Get some Markdown going, find a static generator you like, and save your money for something more interesting—like more hardware for your server rack.

Cheers for stopping by.

— Charlie
