---
# Common-Defined params
title: "Reason to choose Static and Dynamic frameworks in < 5 minutes"
date: "2020-06-19"
description: "Learn how to decide between static or dynamic frameworks"
categories:
  - "iOS Development"
tags:
  - "Swift"
  - "Optimization"
  - "Modularization"

# menu: main # Optional, add page to a menu. Options: main, side, footer

# Theme-Defined params
# thumbnail: "img/wrapper.png" # Thumbnail image
lead: "Learn how to decide between static or dynamic frameworks" # Lead text
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

One day after we have successfully grown our app bigger, we face multiple problems. One problem we found out that our code is super messy and confusing!. Another, why the hell it took a very long time for compiling!. So what is the solution that comes up to our mind? As a beginner, we will think that `Class` separation would be enough for solving the messy code since we know that it is easier to build, less time consuming, and easier to code. How about the second problem? Nah, our sensei in office will suggest `Static` and `Dynamic` framework approach.

Before dive down into both approaches (+ class separation), the question is why do we even need that? So there are multiple reasonable answers for this case, but I will outline 3 answers which are for **code reusability, scalability, and robust development**. By using Microservices approach we basically create good architecture in terms of separation of process. Finally we able to put more weight on our SOLID implementation. Besides that, there are other benefits such as faster launch time, faster compile-time, memory usage, and smaller app size. However, this all depends on what approach we choose. Let us take a look at these comparisons between 3 things

![static-dynamic](/img/static-dynamic-1.png)

Let see the table above. We can see that every single approach has good benefit, it really depends on our use cases. For most of the time if we do struggle with compile-time we will prefer using `Dynamic` frameworks since it is linked during the runtime, and loaded lazily in memory. However, using `Static` approach will give faster launch time on the app itself since everything has been ready because it is embedded in the app beforehand. Moreover using `Class` separation would be our last resort since it is easy to build and less time consuming (easy to set up) for beginners and small products.

![static-dynamic](/img/static-dynamic-2.png)

How about the disadvantages for each approach ?. It is pretty clear that every approach has their own disadvantages. On `Static` approach our binary size can be bloated because everything will be copied together in the app, as well we can not put the Assets into a static approach ([There is a way but with more effort](https://stackoverflow.com/questions/12180113/ios-static-framework-with-resources-inside)). For `Dynamic` approach, the launch time can be slower since we need to link everything first for the first time when the product is being downloaded and opened by the user (For embedded dynamic frameworks), hence [Apple recommends](https://developer.apple.com/documentation/xcode/improving_your_app_s_performance/reducing_your_app_s_launch_time) only limit to 6 to dozens dynamic frameworks per app.

---

Finally, we are able to classify each pros and cons for all approaches, the decision is yours!. Take a look at your use cases, try to define the limitation and the usage of why we would choose one and another before we decide to choose one of them.
