---
# Common-Defined params
title: "Fun Debugging in iOS — Understanding the GUI in 3 minutes"
date: "2021-01-09"
description: "Learn how to use easily debug in Xcode"
categories:
  - "iOS Development"
tags:
  - "Swift"
  - "Debugging"

# menu: main # Optional, add page to a menu. Options: main, side, footer

# Theme-Defined params
# thumbnail: "img/wrapper.png" # Thumbnail image
lead: "Learn how to use easily debug in Xcode" # Lead text
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

After we have understood the basic concept of [LLDB commands](/articles/lldb-basic-concept.md) now it is taking time to understand the full **GUI** of Xcode debugging.

![fun-debugging](/img/fun-debugging-1.png)

Let see the picture above. Once you have put a breakpoint on any lines of code in your applications, you will see a similar screen as above.

1. The top one is our code panel, which indicates where is our breakpoint located

2. The bottom left panel is our quick look panel, we can see our current state of the object in either class or function level at the moment the code is being halted

3. The bottom right panel is our console debugger whereas we can put a lot of LLDB command and expression in there

![fun-debugging-example](/img/fun-debugging-2.png)

When you see on the left panel, you will see a full stack trace of your code. This stack trace is indicating the flow from where it is started until now at the moment the code is being halted. You can click on each line and investigate inside more deeply and analyze each part to identify and gather more results.

![fun-debugging-example](/img/fun-debugging-3.png)

As I said previously, this quick look panel will help us understand more context to all of the current states of the properties in the class and function level before the code is halted. You can click the dropdown arrow to lookup deeper on it, or click the space button or eye button to each property to see as well. We can also identify UIView state in this quick look which is very helpful for **UI Debugging** as well

![fun-debugging-example](/img/fun-debugging-4.png)

1. **This panel**, which is located in the middle, start from left to the right

2. **Pause-continue** button this button is useful for a start, pause and continue the execution of the code after a breakpoint is started

3. **Step over**, this one is very helpful for executing the exact line after the blue breakpoint, so it will not go inside to the function inside in case your breakpoint is stopped at a line of code that pointed to a certain function

4. **Step in**, this one is very helpful for going inside to the certain function that halted because of the blue breakpoint. We can identify more what happens inside that function that lives inside the function (Function inside a function)

5. **Step over**, this one is very helpful if you want to go out from the current function, mostly used after you do step in, and you want to step out to go to another debug process

6. **View Hierarchy debugger**, this one is helpful for capturing the current state of of the UI, and we will get a full stack of layers of our UI

7. **Object Graph Debugger**, this one is helpful for capturing the current memory state of your application at a specific time

![fun-debugging-example](/img/fun-debugging-5.png)

Last but not least this blue panel above is the one for putting other necessary breakpoints (Generic breakpoints) in our Xcode, which I will explain in other articles.
Stay tuned for more articles!
