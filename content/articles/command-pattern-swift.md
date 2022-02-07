---
# Common-Defined params
title: "Command Pattern in real-life"
date: "2021-07-25"
thumbnail: "img/command-pattern/command-pattern.001.jpeg"
description: "Learn How To Use Command Pattern in Swift"
categories:
  - "iOS Development"
tags:
  - "Design Pattern"
  - "Swift"
lead: "Learn How To Use Command Pattern in Swift" # Lead text
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
Recently I have a problem in my team whereas the analytics function become scattered. Some other team create their own function and handling their own caching for tracking the events, and some even create multiple similar function across the modules only for analytics, until then a problem arises.

Since my company heavily depends on **A/B Testing** and data-driven culture, at some point we have found a lot of biased data as well missing parameters, then I start to investigate the issues. Turns out, having scattered places around the codebase that doing a similar thing but ended up having custom logic could break the result calculation. I ended up found a clever and clean way to tidy up this problem, which is **Command Pattern**

# Definition
What is Command Pattern? Command is part of [Behavioral](https://refactoring.guru/design-patterns/behavioral-patterns) design pattern. Behavioral design pattern is design pattern that separating the responsibility between objects to make a more concise and on-target function for each object. Command pattern itself goals are for converting most of the similar or repetitive request functions into a stand-alone object that will become the new place for that converted requests. This conversion will simplify and unified the place of the multiple requests, hence the caller object will not know about its underlying mechanism about how to process those command into the target request. What's the client need is only sending out a simple command request with its parameters. Some enhancement allows you to also add advanced operation such as delay or queue or undo operations.

# Problem

One of the problems I had during the development is how the analytics code is being scattered all over the places like below.

{{< highlight swift "linenos=inline" >}}
final class SomeClass {
  let track: Tracker
  var isCached: Set<String> = .init()

  func handleBuyButton() {
      sendBuyItemClickEvent()
  }

  func sendBuyItemClickEvent() {
      track.trackBuyItemClickEvent()
  }
}
{{< / highlight >}}

As you can see example above is perfectly fine until other users implementing a similar name function with similar action on another class which is tracking buy item events, we will end up with tons of similar functions all over the place. Another concern is we do need to copy the tracking flow in other classes as well. This has become a tedious job for everyone.

# Solution

One of the solutions I am using is using Command pattern. I try to group most of the similar functions within a panel into an enum-based action.

{{< highlight swift "linenos=inline" >}}
enum Action {
  case click
  case seen
}

enum Event {
  case buyItem(action: Action)
}
{{< / highlight >}}

Above is the example of how I divided the enum into two parts, one is the **Action** part and the second one is the **Event** part. However, you can also make it more specific by creating another enum only for a particular screen

{{< highlight swift "linenos=inline" >}}
enum ScreenAEvent {
  case buyItem(action: Action)
}

enum ScreenBEvent {
  case sellItem(action: Action)
}
{{< / highlight >}}

However, there is still a problem that I faced. We need to always **switch case** every action even if the action does not exist with one of the event cases, hence I remove the action enum but put more detailed event definition on the event enum. Another important detail is I need to add the interface for every enum to conform to the parent protocol for all enum, so later we can send this generic protocol declaration instead of a singular specific enum event.

{{< highlight swift "linenos=inline" >}}
protocol ScreenEvent {}

enum ScreenAEvent: ScreenEvent {
  case buyItemClickEvent(itemId: Int)
}

enum ScreenBEvent: ScreenEvent {
  case sellItemSeenEvent(itemId: Int)
}
{{< / highlight >}}

Once the enum is well defined in the interface, I start to create the protocol definition for the **AnalyticsService**. Keep in mind the main function of the service as a command class, is simplifying the way to call the tracking.

{{< highlight swift "linenos=inline" >}}
protocol AnalyticsService {
  var tracked: Set<String> { get }
  func send(event: ScreenEvent)
  func sendOnce(event: ScreenEvent)
}

final class ScreenAAnalyticsServiceImpl: AnalyticsService {
  private(set) var tracked: Set<String> = .init()

  private let tracker: Tracker

  func send(event: ScreenEvent) {
    guard let event = event as? ScreenAEvent else {
      return // or throw error or other approach
    }
    switch event {
      case .buyItemClickEvent(let itemId):
        sendBuyItemClickEvent(id: itemId)
    }
  }

  func sendOnce(event: ScreenEvent) {
    if !tracked.contains("someIdentifier") {
      send(event: event)
      tracked.insert("someIdentifier")
    }
  }

  private func sendBuyItemClickEvent(id: Int) {
    tracker.sendBuyItemClickEvent(id: id)
  }
}
{{< / highlight >}}

Finally, our last step is defining the interface and its implementation class. The class is pretty straightforward, what you can do is implementing both `send` and `sendOnce` as well all of the internal tracking action here.

{{< highlight swift "linenos=inline" >}}
final class SomeClass {

  private let analytics: AnalyticsService

  func handleBuyButton() {
    analytics.send(ScreenAEvent.buyItemClickEvent(itemId: 10))
  }
}
{{< / highlight >}}

Above is the way any caller/class uses the command class which is our analytics service. It is very clean, concise, and easy to use. What the caller needs to do is only injecting the analytics as dependency and call its function with the relevant Event without care about its internal engine. If there is a problem with the analytics then we can narrow down the problem that is happening to the analytics service, not the caller class.

However we can still improve our code, right now it is very tedious to copy-paste most of the function signature of AnalyticsService (`send` and `sendOnce`). One of the major improvements I suggest to do is creating an extension of the protocol to have a default implementation of both and let the implementation class only implement their internal event tracking.

# Conclusion

Command pattern is one of the popular patterns that are easy to implement and help us to do more abstraction to our codebase. We are still able to do more abstraction by making the default extension or even create centralized analytics. What do you think about this pattern? Let me know on [twitter](https://twitter.com/michaelabadiii)!
