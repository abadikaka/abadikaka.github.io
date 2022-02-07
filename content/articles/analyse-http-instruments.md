---
# Common-Defined params
title: "Analyze HTTP Traffic with new HTTP Instrument"
date: "2021-06-19"
description: "Understanding new HTTP Instrument Xcode"
categories:
  - "iOS Development"
tags:
  - "Debugging"
  - "Swift"
lead: "Understanding new HTTP Instrument Xcode" # Lead text
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

WWDC21 has been finished last week with over 200 sessions. Most of the time we heard about [SwiftUI](https://developer.apple.com/videos/play/wwdc2021/10022/) session or about [Async Await](https://developer.apple.com/videos/play/wwdc2021/10132/). However there are also interesting session about debugging that has been described by [Keith Harrison](https://twitter.com/kharrison) [here](https://useyourloaf.com/blog/wwdc-2021-viewing-guide/). In this article I would like to focus on one of the ***underrated*** session which is [Analyze HTTP Traffic in Instruments](https://developer.apple.com/videos/play/wwdc2021/10212/). I also write this article in [WWDC Notes](https://www.wwdcnotes.com/notes/wwdc21/10212/).

> All content copyright © 2012 – 2021 Apple Inc. All rights reserved. Swift, the Swift logo, Swift Playgrounds, Xcode, Instruments, Cocoa Touch, Touch ID, FaceID, iPhone, iPad, Safari, App Store, iPadOS, watchOS, tvOS, Mac, and macOS are trademarks of Apple Inc., registered in the U.S. and other countries. This article is not made by, affiliated with, nor endorsed by Apple.

## Track hierarchy

First is about track hierarchy. The hierarchy consists of 4 important elements :

1. HTTP Traffic instruments : Number of active tasks presented
2. Process : Debuggable process and background tasks daemon
3. Session : Session will shown one track per `URLSession` object. During those session there will be an individual task intervals. Besides that, we can configure the session name for more clarity.

{{< highlight swift "linenos=inline" >}}

let session = URLSession(configuration: .default)
session.sessionDescription = "Main Session" // 👈🏻 Custom session name

{{< / highlight >}}

4. Domain
  - Only task that requested in the domain
	- Give more detail about tasks
	- Showing individual transactions
	- Describing the transaction states

## Task timing

Task timing is also part of the instruments GUI. We will able to see structural timing from the start of the request until the end of the request

{{< highlight swift "linenos=inline" >}}

let task = session.dataTask(with: url) {
	/* handle result */ // 👈🏻 Complete event triggers here
}

task.resume() // 👈🏻 Resume event triggers here
{{< / highlight >}}

## Task Identifier

Besides task timing, we also able to let instrument know to show the task identifier next to the description name. Give a [task identifier][taskIdentifier] for clarity.

{{< highlight swift "linenos=inline" >}}
let task = session.dataTask(with: url) {
	/* handle result */
}
task.taskDescription = "Load Thumbnail"
task.resume()
task.taskIdentifier // 👈🏻 this identifier will be shown in instruments
{{< / highlight >}}

## Task Error

Task error is giving an ability for us to distinguish error and success with a color in the Instruments (normally red for an error)

![Task Error][6]

## Transactions

Transactions are part of how instrument is breaking down the whole task timing. Request and response pair is being handled by the URL Loading system by default. Transactions contains all the HTTP layer information such as `URL`, `HTTP Version`, `Connection` and `cache info`. They also have information regarding `Request and Response header` alongside with the `Request and Response body`.

![Transactions][7]

## Transaction states

Another important chunk of information in Instruments are there are multiple states within the transaction for distinction between each block that described in each state. Each state also showing different timing on each layers.

![States][8]

## Instrument

![Instrument][10]

Next there are several step that we can do with the HTTP instrument:

1. Focus on the task description
- Here we can just click on the task that we want, especially if we have already set the description name we can click on it.

![Focus on the task description][11]

2. Expand the task description
- We can expand the specific task description for getting more detailed information on that specific tasks. We also able to drag our pointer to specific timeline to show more compact version of it.

![Expand task description][12]

3. Filter by connection
- Another great utilities are how we are able to filter the task by the connection instead of the tasks.

![Filter task][13]

4. Identify the http task issues (Ex: Staircase problem)
- We can also easily identify the task issues by see the pattern of the connection for each transaction. By an example is a staircase problem.

![Problem][14]

5. Identify the request (Ex: wrong expiration date of cookie)
- Once we are able to identify the issues we can start to identify the request that we want to see. Click on the request, then filter by transaction list on the bottom left filter, and then you will see on the bottom right of the request information (extended detail view), and we can start reading each information. In this case, we found out the wrong expiration date of the cookie.

![Problem 1][15]

6. Identify the backtrace (Ex: cached response)
- We can also to check the backtrace of the problem. The same step from before, however change back the list into URLSession Tasks, and on the bottom right you will see the backtrace and start identify each trace.

![Problem 2][16]

7. Filter the session (Ex: Found an issue with the data that sent to server)
- As I have described on the step 5 and 6 we can filter out the session by the URLSession Tasks, Transaction List or Summary Transactions Duration.

![Problem 3][17]

8. Export the report to .har extension
- One last unique thing is we can export our report to an `.har` extension. First we need to save our trace by clicking `File -> Save As -> Save`. Then we can use a command line to export it as `.har` extension.

```
xctrace export --input YourTrace.trace --har
```

9. Output
- Save and export report to `.har` to analyze the issues. `.har` is a JSON structure so it is very convenient for reading the structure of the report.

```
{
  "log": {
    "version": "1.2",
    "creator": {
      "name": "xctrace",
      "version": "13.0"
    },
    "entries": [
    {
      "_transactionUUID": "BBSA-XXX",
      "_taskUUID: "0293-2093",
      "serverIPAddress": "182.19.40.111",
      "startedDateTime": "2021-05-27T09",
      // and more
    }
     // This JSON contains most of the trace report
    ]
  }
}
```

## Conclusion

As conclusion, this session is really interesting session how Apple provide us more tools to help our development process without necessity of other third party tools for identifying bug of an HTTP Request. I would love to see how it goes and impact most of the team to reduce the amount of the bug that occur during a networking process.

Some tips from the WWDC21 10212 Session:
- Target app  today to detect problems
- Name your `URL Session` and `URLSessionTask` for easier debugging
- Adopt latest networking protocols
- Audit your app requests to check if it can send less information

[6]: /img/wwdc21/10212/6.png
[7]: /img/wwdc21/10212/7.png
[8]: /img/wwdc21/10212/8.png
[10]: /img/wwdc21/10212/10.png
[11]: /img/wwdc21/10212/11.png
[12]: /img/wwdc21/10212/12.png
[13]: /img/wwdc21/10212/13.png
[14]: /img/wwdc21/10212/14.png
[15]: /img/wwdc21/10212/15.png
[16]: /img/wwdc21/10212/16.png
[17]: /img/wwdc21/10212/17.png
[18]: /img/wwdc21/10212/18.png
[19]: /img/wwdc21/10212/19.png

[taskIdentifier]: https://developer.apple.com/documentation/foundation/nsurlsessiontask/1411231-taskidentifier
