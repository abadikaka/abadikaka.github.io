---
# Common-Defined params
title: "Fun Scripting in Swift — for beginner (PART I)"
date: "2020-06-17"
description: "Implementing Argument Parser in Swift"
categories:
  - "iOS Development"
  - "Backend Development"
tags:
  - "Swift"

lead: Implementing Argument Parser in Swift" # Lead text
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

Have you ever used fastlane? swiftlint? xcodegen?. Yeah as an iOS developer we must be familiar with all of those tools for doing automation stuff. How about we need something that no one provides for us? Well, we can make one of those scripts by using Swift!. First of all, if you haven’t watched a wonderful [UIKonf](https://www.youtube.com/watch?v=KbwmvThZUDM) conference featuring [Federico Zanetello](https://www.fivestars.blog/), you need to put it on your watchlist !.
In this article, I would like to explain how does swift scripting is working also by demonstrating the usage of it. I will divide into 2 articles, this one will cover the basic knowledge of Swift Scripting and the second part will showcase the result of the outcome.

In this article, I would like to explain how does swift scripting is working also by demonstrating the usage of it. I will divide into 2 articles, this one will cover the basic knowledge of Swift Scripting and the second part will showcase the result of the outcome.

> Q: What will the outcome of this article? — Good question! We will make a script that allows the user to make a new directory and moving it into a destination path, sounds great eh ?!!

Okay first of all let us start with the setup. Create a new executable project from terminal using Swift Package Manager.

```
mkdir iOSScripting
cd iOSScripting
swift package init --type executable
```

From here you will get a whole package structure inside the folder. You can open and run this project using XCode or just using command `swift build <projectname>` `swift run <projectname>` or `swift test`.

![scripting](/img/scripting-part1-1.png)

How do we pass arguments to the script? First of all, go to the Sources folder and open your main.swift file. There are multiple ways to pass an argument to the script. I will discuss three ways to distribute it

- Through CommandLine arguments
- Through readLine
- Through Environment Variables

First of all using CommandLine arguments are the easiest way to use. Look at my example below. The first argument will be always the execution path so we do need to drop it first from the array.

![scripting](/img/scripting-part1-2.png)

Now to make our script more interactive we need to have 2-way communications, and we can use `readLine()` command which is a synchronous command.

![scripting](/img/scripting-part1-3.png)

Let's go to the third option!. The third option is using Environment Variables. Yep as a developer, we will familiar with this option. Use the `ProcessInfo` API provided by Apple, we can get the value from the environment which we define beforehand from the arguments and retrieve it by using the key we set.

![scripting](/img/scripting-part1-4.png)

Another key point in scripting is we can use a `Pipeline` concept. Think like a flow that there is a connection between one and another. In scripting, we can use the first output as the input of a second command, and second output as the input of the third command and so on. We can use FileHandle API for handling the `Pipeline` concept.

```
let standardInput: FileHandle = .standardInput
let data = standardInput.availableData // To take the available data
if let inputString = String(data: data, encoding: .utf8) {
  print("Data \(inputString)")
}
usage :
$ printf "Michael" | swift run iOSScripting
// You can use another command such as echo or anything that produce
// an output and use Pipe for taking the output as an input on
// second command.
```

So that is the basic knowledge about Swift scripting. There are still a lot more things to explore such as ArgumentParser and using Swift Tool Support for adding animation into our script which all of these topics will be covered in the next article together with our outcome and the way to release our scripts !. Stay tuned for the next article which we will start to make a new script as I promise.
