---
# Common-Defined params
title: "Introducing SFSymbolsFinder"
date: "2020-12-31"
description: "Compact Tools for SFSymbols"
categories:
  - "Open Source"
tags:
  - "Library"

# menu: main # Optional, add page to a menu. Options: main, side, footer

# Theme-Defined params
# thumbnail: "img/wrapper.png" # Thumbnail image
lead: "Compact Tools for SFSymbols" # Lead text
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

Recently I got stumbled with a boring way to create an icon from **SF Symbols**. As you know that Apple introduces SF Symbols for a developer to reuse in their own application if needed. SF Symbols is a set of over 1,500 symbols that you can use in your app. Why **SFSymbols** are great for indie developers?

1. It is free
2. Optical alignment with text for all weights and sizes
3. Support multicolor
4. Support localization for RTL languages such as Arabic, Devanagari, and Hebrew

Normally we will do something like this for using the SFSymbols

```
UIImage(systemName: "gear") --> UIKit
// or
Image(systemName: "gear") --> SwiftUI
```

But it is quite burdensome if I need to do it for a longer string such as `line.horizontal.3.decreasesymbol`. Since I need to apply some of the icons in my own pet project, I decided to make a new Swift Package Manager for this one.

## SFSymbolsFinder

**[SFSymbolsFinder](https://github.com/abadikaka/SFSymbolsFinder)** will act as convenient enum that converts available symbols to either Image or a String. This library will support more than 15 available categories listed in the SF Symbols app from Apple. Right now it is mostly supported symbols from v1.1, however gradually I will add more support for v2.2
How to use these symbols? It is pretty easy. Considering we want to use SwiftUI in our project, we can either use an image or string of system name like below

{{< highlight swift "linenos=inline" >}}
import SwiftUI
import SFSymbolsFinder

// Use Shape Circle symbols

Shape.rectangle.image // return Image(systemName: "rectangle")

Shape.rectangle // return a View

Image(systemName: Shape.rectangle.systemName) // return "rectangle"

// Use All for every SF Symbols

All.rectangle.image // return Image(systemName: "rectangle")

All.rectangle // return a View
{{< / highlight >}}

You can either use one of the options as I described above
The benefit of using SFSymbolsFinder:
- You don't need to type a literal string every time
- You have autocompleted for each category when typing in the code
- You can have one single source of truth where the Image is coming from
- All icon name and enum is matched with the one provided with apple

Finally, that’s all my introduction to my latest SPM package. Find out more about the official documentation here: https://github.com/abadikaka/SFSymbolsFinder
All contributions and supports are greatly appreciated!
