---
# Common-Defined params
title: "Understanding Proxy Pattern in Swift within 5 minutes"
date: "2021-01-26"
description: "Implementing Proxy Pattern in Swift"
categories:
  - "iOS Development"
tags:
  - "Design Pattern"
  - "Swift"
lead: "Implementing Proxy Pattern in Swift" # Lead text
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

The proxy pattern is one of the few underrated patterns that nobody really talks about in iOS. A proxy pattern is one of the structural design patterns that lets you provide a substitute or placeholder for another object. A proxy has total control and access to the original object, and let the proxy object do some magical stuff with the original object and restrict the access to it, however, the one who uses a proxy object, won’t have any knowledge behind the scene.

Proxy pattern normally being used outside of mobile development, when some request wants to map the request to a specific object without exposing underlying knowledge about that object, so it can be interchangeable with any object that conforms to the original object. However, proxy patterns can be also used in mobile development.

![proxy](/img/proxy-pattern-1.png)

The benefit of the proxy, it can be disguised as a real object, because the interface is similar to the real client object, however, we don’t interrupt the original object that will do the process under the hood, which means we can interpolate or dislodge something in the proxy without we harm the original object. Proxy is having similar behavior as a gateway to the original process of the operation.

## USE CASE

Let’s take real use case here. A repository object wants to call a network request to an API, however since the company has moved forward to the new system, they want to migrate gradually from the old REST API to the GraphQL endpoint. So in order for the repository to still able to work with the existing interface without breaking change, we will make a Proxy object that has a similar interface as the original object does


{{< highlight swift "linenos=inline" >}}
protocol Requestable {
  typealias Params
  func fetchData(with params: Params)
  func updateData(with params: Params)
}

final class Proxy: Requestable {
  typealias Params = [String: Any]

  // 1. Your service must be private
  // TIPS: You can use enum and factory to create service base on the type
  private lazy var service: Requestable = LegacyRestService() // or GraphQLService

  func fetchData(with params: Params) {
    // do something with your parameters or other object
    // if you wish before making real request

    service.fetchData(with: params)    
  }

  func updateData(with params: Params) {
    service.updateData(with: params)
  }

}
{{< / highlight >}}

So the code is quite simple, define your interface of the service. Then make your real implementation class of the service for both classes that serving the old system with REST API and the new system with GraphQL. This how the caller looks like. Pretty simple, instantiate your proxy class and just call the proxy method.

{{< highlight swift "linenos=inline" >}}
final class ProxyImplementation {
  private lazy var proxy: Proxy = Proxy()
  func refreshPage() {
    proxy.fetchData(with: ["page": 1])
  }

  func updatePage(status: Bool) {
    proxy.updateData(with: ["update": status])
  }
}
{{< / highlight >}}

There is some confusion related to the difference between an adapter and a proxy pattern. This stack overflow [answer](https://stackoverflow.com/questions/37692814/what-is-the-exact-difference-between-adapter-and-proxy-patterns) has a very good answer about that. Basically Proxy is only a surrogate and it has a similar interface with the original object, however the opposite with the adapter. The adapter pattern's intention is to help the accessor able to work with the old system that can’t work together with the new implementation, hence the adapter object and the adaptee object has a different interface.
