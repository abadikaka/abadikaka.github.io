---
# Common-Defined params
title: "Leverage Accessibility Identifier For View Debugging"
date: "2021-10-02"
thumbnail: "img/accessibility/accessibility.001.jpeg"
description: "Learn how to debug your view with an accessibilityId"
categories:
  - "Automation"
  - "iOS Development"
tags:
  - "Accessibility"
  - "SwiftUI"
  - "Testing"
  - "UIKit"
lead: "Learn how to debug your view with an accessibilityId" # Lead text
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

Recently I have been fascinated with the fact many people, especially indie dev have started to think about accessibility for their application. However, there are still fewer resources on how do we kick off our journey in Accessibility with a more real-world use case that we can adapt. In this article, I would like to explain how do we utilize and incorporate the most basic Accessibility topics into our codebase, which is `accessibilityId`.

{{< sponsor >}}

## Definition

What is **Accessibility**? Accessibility, also referred to as inclusivity, is how do we able to create an application that can be used widely for everyone, not limited to any specific groups. By it means, Accessibility will help people that fall under people with disabilities and special needs to also have access and be comfortable in using the app we create.

In SwiftUI, it comes with built-in accessibility support that is easily attached to each SwiftUI element such as view, list, text, label, button, and so on. There are also several modifiers to support features like **VoiceOver**, **Voice Control**, and **Switch Control** like `accessibilityLabel` and `accessibilityValue`. There is also a special modifier for setting the `id` for each given view which is `accessibilityId` that also is being used for doing an **Automation Test** using `XCUITest`.

## Problem

Imagine you are working with a big company that has hundreds of developers working on the same codebase. Many people come and left every year, which left out some code for us to take over. One day, we found out a bug somewhere in that particular view, but we don't have much knowledge of where does this bug is coming from? Which view is being affected? and so on. Or even worse, there is a flaky UI Test or suddenly our code makes the AT did not pass. Here `accessibility` features and tools can help us to debug the view hierarchy much faster. In this article, I will explain the comparison of using `accessibility` features in both **UIKit** and **SwiftUI** able to help us in a different way.

## UIKit, View Hierarchy and Accessibility

When developing an application with UIKit we often need to debug our UI with a view hierarchy to get an overview of the current snapshot of the app at the exact point of the time it was captured.

![example](/img/accessibility/example1.png)

In the above example, we can see the current state of our view hierarchy, and we can easily navigate to the view we want to observe. If we are setting the `accessibilityId` of the view, you can debug it easier by clicking on the related view and take a look at the right panel and you can see the value of the `accessibilityId` in there or other Accessibility attributes, then you can easily find the code of that particular view that you want to debug. No more hassle for your future self or other developers that go to the same problem.

![example](/img/accessibility/example2.png)

## SwiftUI, View Hierarchy and Accessibility

Now let's take a look on how we do set `accessibilityId` in SwiftUI.

{{< highlight swift "linenos=inline" >}}
VStack {
  Text("My Text")
  .accessibilityIdentifier("My_Text_Identifier")
}
{{< / highlight >}}

As you can see it is pretty easy to set the id into any of your views that you want to set. Now let's capture the view hierarchy and see below


![example](/img/accessibility/example3.png)

**Daang!!**, here we can see the first problem, our view hierarchy does not clearly explain what kind of view of each element in the stack hierarchy. In fact, the naming is different than the one we write in the code, and lot of unfamiliar syntaxes, why does this happen? As for now, I don't have a clear answer for this, but my assumption is because the layout system is different hence the view hierarchy is not a good solution to see the visual breakdown of our SwiftUI application. Then how about our `accessibilityId` that we have set? Can we still see the value?

![example](/img/accessibility/example4.png)

**Bad news!**, even the `accessibilityId` in the view that we select does not even show in the right panel inspector. There is the Accessibility attributes section, but it doesn't help much because the value inside is not explicitly telling us the `identifier`.

Ok, so how we really can utilize our `accessibilityId` in SwiftUI for debugging purposes? There is one trick from me, use `Accessibility inspector`!. Now head over your Xcode, and open the developer tools and select `Accessibility inspector`. **Accessibility Inspector** is a developer tool from Apple that let us debug the **Accessibility features** (not only `accessibilityId`, but also label and voice and dynamic type) on any target that we want to observe, in this case our application through the **Simulator**.

![example](/img/accessibility/example5.png)

Now you see these tools and you can start running your app and click any view that you want to inspect. Once you click on it, you can see the detail of all of the **accessibility features** in the selected view. Finally, you can see your `accessibilityId` and can start to find the code in the codebase!

![example](/img/accessibility/example6.png)

## Caveat

However, there is a caveat to using `Accessibility inspector`. This tool won't solve your existing problem if the previous coder did not put the `accessibilityId` into the view, hence this is could be a good chance for you to redeem his/her fault and help the future developer or even yourself. Anyway putting an `accessibilityId` is always good practice for starting point to Automation Test and even to start thinking about **Accessibility** on your app.

## Conclusion

[**Accessibility**](https://developer.apple.com/documentation/swiftui/view-accessibility) is indeed an important factor for enhancing our application to be more accessible, however, we can do more than it. We can leverage the usage of an `accessibilityId` even if you haven't started to think about **Accessibility**, by using it for debugging purposes. Also, we want to help ourselves and future developers as good citizens in iOS development. Do you agree with my solution? Or do you have any feedback about this article? Reach me out on [twitter](https://twitter.com/michaelabadiii)!

## Good Resources

Last but not least, I have recently started to take a look to bring my app to the next level by thinking of [Accessibility](https://developer.apple.com/documentation/swiftui/view-accessibility). One of the good resources to start is by reading accessibility articles from [Majid](https://swiftwithmajid.com/). So head over to his article and enjoy your reading!

{{< sponsor2 >}}
