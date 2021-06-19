---
# Common-Defined params
title: "Symbolic Breakpoint Xcode in 2 Minutes"
date: "2020-11-09"
description: "Use Symbolic Breakpoint for debugging"
categories:
  - "iOS Development"
tags:
  - "Testing"
  - "Swift"
lead: "Use Symbolic Breakpoint for debugging" # Lead text
comments: false # Enable Disqus comments for specific page
authorbox: true # Enable authorbox for specific page
pager: true # Enable pager navigation (prev/next) for specific page
toc: true # Enable Table of Contents for specific page
mathjax: true # Enable MathJax for specific page
sidebar: "right" # Enable sidebar (on the right side) per page
widgets: # Enable sidebar widgets in given order per page
  - "search"
  - "recent"
  - "taglist"
  - "social"
draft: false
---

Once upon a time, while I was debugging my code, I found out an annoying warning comes up like below

```
[TableView] Warning once only: UITableView was told to layout its visible cells and other contents without being in the view hierarchy (the table view or one of its superviews has not been added to a window). This may cause bugs by forcing views inside the table view to load and perform layout without accurate information (e.g. table view bounds, trait collection, layout margins, safe area insets, etc), and will also cause unnecessary performance overhead due to extra layout passes. Make a symbolic breakpoint at UITableViewAlertForLayoutOutsideViewHierarchy to catch this in the debugger and see what caused this to occur, so you can avoid this action altogether if possible, or defer it until the table view has been added to a window.
```

There was no fixed solution out there on how to resolve this and what kind of things that cause this to happen. One thing I could understand, is the UITableView is forced to layout-ing its visible cells but actually, they were not in the view hierarchy yet !. What was the possible impact if we annoy this bug?

1. Performance, performance, and performance while UIKit draws our views

2. Undetected weird glitch may occur

3. Fighting with your colleagues. UPS, Kidding lol

How to solve this issue then? Pretty easy, read all of the available logs provided by the compiler, pay attention to the last sentence. Explicitly, Xcode has given us the solution.

```
Make a symbolic breakpoint at UITableViewAlertForLayoutOutsideViewHierarchy to catch this in the debugger and see what caused this to occur, so you can avoid this action altogether if possible, or defer it until the table view has been added to a window.
```

Yes that’s it lets turn on our **Symbolic breakpoint**. Symbolic breakpoints are breakpoints that will trigger on certain conditions. How to do that?

1. Navigate to the second last tab on the left panel with a breakpoint picture.
2. Click the “+” icon at the lower left of the Breakpoint Navigator and select **Symbolic Breakpoint**.

![symbolic-breakpoint](/img/breakpoint-1.png)

After you finally click on it, fill in the name and the symbol. There are lots of other important symbols that Apple has, but for the sake of this debugging let’s put what Apple wants for our case

![symbolic-breakpoint](/img/breakpoint-2.png)

Now let’s run your project, anytime that the project nearly produces the warning, it will call the breakpoints, and you will get the stack trace and identify your problem. In my case, it was because I forced the view to layout the view if needed while the table view hasn’t been ready.

Good luck!
