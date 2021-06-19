---
# Common-Defined params
title: "Simple Slick SwiftUI Onboarding View"
date: "2021-04-08"
description: "Learn how to use TabView for onboarding in SwiftUI"
categories:
  - "iOS Development"
tags:
  - "SwiftUI"
# menu: main # Optional, add page to a menu. Options: main, side, footer

# Theme-Defined params
# thumbnail: "img/wrapper.png" # Thumbnail image
lead: "Learn how to use TabView for onboarding in SwiftUI" # Lead text
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

Apparently, even if SwiftUI 2.0 has powerful enough for production code, but sometimes they are still lacking documentation for a simple component like PagingView. As your information, there is now a native equivalent of `UIPageViewController` in SwiftUI 2.0 which is adding the `.tabViewStyle` modifier to `TabView` and pass `PageTabViewStyle`

![simple-slick](/img/simple-slick.png)

Let me go through step by step.

- We will need to wrap our view with the **GeometryReader** in order to get the parent relative size position for configuring the onboarding view frame.

- **VStack** is not necessary, in here I put it first because I will show you that we could animate some buttons later base on the view for each onboarding.

- **TabView** is the important part here. We can use **TabView** for simulating our onboarding and we also can give the index of the current view position. Don’t forget to add the **tag** for each embedded onboarding view. The **tag** is for the identifier of the view that will become the selection state that we set on **TabView**. Every time the bindable selection state is changing, it will read the tag to get the correct identifier for the current view that needs to be displayed.

- **TabViewStyle** modifier, we need to set our style to be **PageTabViewStyle** which is a new modifier for a paging view style on **TabView**.

- **IndexViewStyle** is also important as it is the styling for the circle indicator. By default, it has a white background, so it won’t be visible unless we change our view background color. However, we can set it to always so we have a nice neat standard iOS Paging View indicator style

- Now set up our view frame base on our parent size

Once you have done all of the steps above you will get some result like below

![slick-ob](/img/slick-ob-1.gif)

However, there is still some weird feeling. There is no smooth animation when swiping the view, and every view has the same white background. So now let’s try to have different colors for each view. See the code below.

{{< highlight swift "linenos=inline" >}}
private func switchColor() -> Color {
  let tabColor = TabColor(rawValue: currentIndex) ?? .one
  return tabColor
}

GeometryReader(content: { geometry in
  VStack {
    // .. //
  }
})
.background(
  switchColor()
    .edgesIgnoringSafeArea(.all)
)
.animation(.easeInOut(duration: 0.2))
{{< / highlight >}}

1. We made `switchColor` function which generates a color for each view depend on the currentIndex

2. In the **GeometryReader** set your background and we want to ignore all of the safe areas

3. Then simply add `.animation` here and you can choose any of the animation styles you wish, in this case, I pick `.easeInOut`

Now you will see a better onboarding view like below

![slick-ob](/img/slick-ob-2.gif)

Let’s add one more cool thing to our onboarding view, which is adding the next button and back button, as well as the free animation from SwiftUI for free. See the snippet below

{{< highlight swift "linenos=inline" >}}
GeometryReader(content: { geometry in
  VStack {
      // .. //
      HStack(alignment: .center, spacing: 8) {
        if currentIndex > 0 {
          Button(action: {
            currentIndex -= 1
          }) {
            HStack(alignment: .center, spacing: 10) {
              Text("Kembali")
                .foregroundColor(ColorStyle.emerald.color)
            }
          }
          .frame(
              minWidth: 0,
              maxWidth: .infinity,
              maxHeight: 44
          )
          .background(Color.white)
          .cornerRadius(4)
          .padding(
              [.leading, .trailing], 20
          )
        }
        Button(action: {
          if currentIndex != 2 {
            currentIndex += 1
          }
        }) {
          HStack(alignment: .center, spacing: 10) {
            Text(currentIndex == 2 ? "Mulai" : "Lanjut")
              .foregroundColor(.white)
              All.arrowRight
                .accentColor(.white)
          }
        }
        .frame(
            minWidth: 0,
            maxWidth: .infinity,
            maxHeight: 44
        )
        .background(ColorStyle.emerald)
        .cornerRadius(4)
        .padding(
            [.leading, .trailing], 20
        )
      }
      Spacer()
    }
  }
})
.background(
  switchColor()
    .edgesIgnoringSafeArea(.all)
)
.animation(.easeInOut(duration: 0.2))
{{< / highlight >}}

1. Let’s add 2 Buttons to our view. The first button is the previous button and the second button is the next button with an arrow inside the button. Since there are 2 buttons then we will use HStack to wrap it.

2. We will put logic to our button. The first button only showing if the index is not 0 and we will decrement the currentIndex value by one every time we click on it.

3. The second button we always showing it however there are 2 different texts, if it is the first index then it will have “Start” text and “Continue” for the second afterward. We also increment the currentIndex value by one every time we click on the button.

4. Then we can put some styling like background, corner radius, padding, height, and width. Let’s put .infinity for maxWidth so the button will fill up the available space on the HStack.

Now you will see the amazing result like below

![slick-ob-3](/img/slick-ob-3.gif)

Easy right? So now it is your turn for creating the amazing onboarding view for your application!
