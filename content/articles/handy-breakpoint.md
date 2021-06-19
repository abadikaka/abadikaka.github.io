---
# Common-Defined params
title: "Xcode handy “Move and Share” breakpoint in 2 minutes"
date: "2020-12-27"
description: "Learn how to use handy debugger in Xcode"
categories:
  - "iOS Development"
tags:
  - "Swift"
  - "Debugging"

# menu: main # Optional, add page to a menu. Options: main, side, footer

# Theme-Defined params
# thumbnail: "img/wrapper.png" # Thumbnail image
lead: "Learn how to easily debug in Xcode" # Lead text
comments: false # Enable Disqus comments for specific page
authorbox: true # Enable authorbox for specific page
pager: true # Enable pager navigation (prev/next) for specific page
toc: false # Enable Table of Contents for specific page
mathjax: true # Enable MathJax for specific page
sidebar: "right" # Enable sidebar (on the right side) per page
widgets: # Enable sidebar widgets in given order per page
  - "search"
  - "recent"
  - "taglist"
  - "social"
draft: false
---

Do you know that we can actually simply share our breakpoints with our colleagues at work? Xcode provides us with the capability to share it. This share function is very handy if we want all colleagues to have a similar breakpoint. Some breakpoints that worth sharing are Exception breakpoint, Runtime breakpoint, and some Symbolic breakpoints. Let see how it works

1. Go to your debug navigator panel on the left panel

2. Add your breakpoint and choose any of it

3. Right-click on it and choose to “Share” breakpoint

![fun-debugging-example](/img/handy-debug-1.png)

We also have another handy feature, the same as above but we choose another option which is a move to “User” options. What are the advantages of this one? This one will help us to open any Xcode project with a preselected breakpoint of our configurations, so no need to do duplications of work to add a breakpoint. This one is very helpful if we want to consistently have a similar breakpoint for all future projects.

![fun-debugging-example](/img/handy-debug-2.png)

Another interesting thing is we can do exactly to add more interesting action on our individual breakpoint. Simply click on the breakpoint and add multiple conditions such as name, condition, ignorance counter, action, and ability to tick the option to ignore the breakpoint to halt the execution.

![fun-debugging-example](/img/handy-debug-3.png)

There are multiple interesting actions that we can apply such as adding a sound or even a script, this one very handy for game development or identify memory leak on deinit with a sound and we can ignore the breakpoint execution to be stopped.

![fun-debugging-example](/img/handy-debug-4.png)

We can utilize our breakpoint to be more precise and useful for our application debugging process. Even we can add more handy features such as share or move our several breakpoints. Stay tuned for the next articles!
