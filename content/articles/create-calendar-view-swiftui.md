---
# Common-Defined params
title: "Create Draggable Calendar View in SwiftUI"
date: "2021-09-12"
thumbnail: "img/calendar/calendar.001.jpeg"
description: "Learn how to use onDrag and onDrop for creating a calendar view"
categories:
  - "iOS Development"
tags:
  - "SwiftUI"
lead: "Learn how to use onDrag and onDrop for creating a calendar view" # Lead text
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

While I was looking at my twitter for inspiration, a tweet from [Sean Allen](https://twitter.com/seanallen_dev) caught my [eyes](https://twitter.com/seanallen_dev/status/1431299626137759749?s=20). He made an amazing calendar component with a drag and drop capabilities of its item. I look up the possibilities of using the native SwiftUI approach without any UIKit hack or workaround to do so. In this article, I will explain step by step as well a brief explanation of the API I used to make the calendar view in SwiftUI. There is a good [stackoverflow](
https://stackoverflow.com/questions/62606907/swiftui-using-ondrag-and-ondrop-to-reorder-items-within-one-single-lazygrid) answer to implement drag and drop on 1 level item, but it could not work in case we need to drag and drop the item of the parent item to another parent item, hence I created a workable solution to solve that problem.

{{< sponsor >}}

## Construct the data

First, we do need basic data to start with. We only need two models in this use case, a `ScheduleItem` and a `CalendarViewModel`. The `ScheduleItem` is representing the sub-item of the calendar that we want to drag and drop, but the `CalendarViewModel` is the model that we want to place within the `Calendar` view, hence we do need around 30/31 `CalendarViewModel` per month.

{{< highlight swift "linenos=inline" >}}

struct ScheduleItem: Identifiable, Equatable {
    let id: String = UUID().uuidString
    let name: String? // I will explain later why we need an optional name. This is part of the fake concept I will explain later.
}

struct CalendarViewModel: Identifiable, Equatable {
    let id: String = UUID().uuidString
    let date: String
    var items: [ScheduleItem] // I make it var because we will mutate this later
}

{{< / highlight >}}

## Layouting the Calendar Grid and Calendar Items

In this tutorial, I will create a lite version of a calendar view. We only need a `LazyVGrid` with 7 columns that represents 7 days per week. We also will create a collection of `CalendarViewModel` as a `@State` and also one of the `ScheduleItem` as a `dragged` object.

{{< highlight swift "linenos=inline" >}}
struct CalendarView: View {
    @State private var models: [CalendarViewModel] = []
    @State private var dragging: ScheduleItem?

    private let sevenColumnGrid = Array(repeating: GridItem(.flexible(), spacing: 0), count: 7)

    var body: some View {
        VStack {          
              LazyVGrid(columns: sevenColumnGrid) {
                  // ** 2. items ** //
              }
              .animation(.default, value: models)
        }
        .background(Color.white)
    }
}
{{< / highlight >}}

In the code below, this is the part where we are layouting the `Item` view. We only need to iterate the calendar items and make a `View` on top of it. If you notice you will see I made two separate views, one is the view of the item with a name is exist and the other one is the view for the blank item. I put this way instead of a blank `Spacer` because of a reason that I will explain in the caveat section later.

{{< highlight swift "linenos=inline" >}}
// 2. items put this inside the LazyVGrid

ForEach(models) { model in
    VStack {
        Text(model.date)
            .frame(maxWidth: .infinity, alignment: .trailing)
        ForEach(model.items) { item in
          if let name = item.name {
              VStack {
                  Spacer()
                  Text(item.name)
                      .frame(maxWidth: .infinity, alignment: .center)
                  Spacer()
              }
              .background(Color.green.opacity(0.3))
              .cornerRadius(4)              
          } else {
              VStack {
                  Spacer()
                  Text("")
                      .frame(maxWidth: .infinity, alignment: .center)
                  Spacer()
              }              
              .background(Color.white)              
          }          
        }
        if model.items.isEmpty {
            Spacer()
        }
    }
    .frame(minWidth: 0, maxWidth: .infinity, minHeight: 100, alignment: .center)
}
{{< / highlight >}}

The calendar view now will be look like below!.

![calendar](/img/calendar/calendarView.png)

## Understanding onDrag and onDrop API

What is `onDrag` ? `onDrag` is a method that is being used for activating the view as the initial source of drag and drop operation, which means it will return `View` that is being attached with the user gesture input by the time the method is being called. Every time the drag operation begins, the attached view will be used as the preview image. `onDrag` modifier has a closure that will create and return `NSItemProvider`. `NSItemProvider` is the class that informs the system about the content and the type of the draggable item, in this case, we force it as `NSString` since the `UTType` we choose is .text (We use `NSString` not `String`, because `NSItemProvider` is part of `NSObject`, which indicates it can behave as an **Objective-C** objects so we do need to inject a `String` data type that could be used in Objective-C which is `NSString`).

How about `onDrop`? So basically `onDrop` will ask for a `UTType` data type which is a data type that represents a type of data to load, send, or receive. [UTType](https://developer.apple.com/documentation/uniformtypeidentifiers/uttype) data type describes type information of data such as a unique identifier, lookup method name, or even some of the metadata as part of additional information to be sent along with the data. In this case, I will just use `UTType.text` type because we just drop simple text data from a struct.

So after all of the theory above, here is the implementation of the `onDrag` and `onDrop` on our calendar view. (Put this inside the `ForEach` of the Grid).

{{< highlight swift "linenos=inline" >}}

VStack {
    Text(model.date)
        .frame(maxWidth: .infinity, alignment: .trailing)
    ForEach(model.items) { item in
        if let name = item.name {
            VStack {
                Spacer()
                Text(name)
                    .frame(maxWidth: .infinity, alignment: .center)
                Spacer()
            }
            .background(Color.green.opacity(0.3))
            .cornerRadius(4)
            .overlay(dragging?.id == item.id ? Color.green.opacity(0.3) : Color.clear)
            .onDrag {
                self.dragging = item
                return NSItemProvider(object: item.id as NSString)
            }
            .onDrop(of: [UTType.text],
                    delegate: DropDelegateImpl(item: item,
                                               listData: $models,
                                               current: $dragging))
        } else {
            VStack {
                Spacer()
                Text("")
                    .frame(maxWidth: .infinity, alignment: .center)
                Spacer()
            }
            .background(Color.white)
            .onDrop(of: [UTType.text],
                    delegate: DropDelegateImpl(item: item,
                                               listData: $models,
                                               current: $dragging))
        }
    }
    if model.items.isEmpty {
        Spacer()
    }
}
.frame(minWidth: 0, maxWidth: .infinity, minHeight: 100, alignment: .center)

{{< / highlight >}}

{{< sponsor >}}

## Understanding DropDelegate

For both `onDrag` and `onDrop` are working as expected, they require us to implement [`DropDelegate`](https://developer.apple.com/documentation/swiftui/dropdelegate) on the `onDrop` behavior. The purpose of `DropDelegate` is to give us the ability to interact with a drop operation in a view modified to accept drops. If we don't need to do complex behavior or interaction then you can proceed with another `onDrop` modifier which has a `perform` closure for doing the drop interaction. In this case, since we do need to detect where the item is coming from and to whom the item is expected to be dropped, as well as performing a swap operation we need a custom `DropDelegate`. This `DropDelegate` will be applied to all of the calendar items as part of a parameter of view modifier of `onDrop` modifier.

{{< highlight swift "linenos=inline" >}}

struct DropDelegateImpl: DropDelegate {
    let item: ScheduleItem // 0.
    @Binding var listData: [CalendarViewModel] // 1.
    @Binding var current: ScheduleItem? // 2.

    func dropEntered(info: DropInfo) {
        guard let current = current, item != current else {
            return
        } // 3.
        let from = listData.first { cvm in
            return cvm.items.contains(current)
        } // 4.
        let to = listData.first { cvm in
            return cvm.items.contains(item)
        } // 5.

        guard var from = from, var to = to, from != to else {
            return
        } // 6.

        if let toItems = to.items.first(where: { $0.id == item.id }),
           toItems.id != current.id { // 7.
            let fromIndex = listData.firstIndex(of: from) ?? 0 // 8.
            let toIndex = listData.firstIndex(of: to) ?? 0 // 9.
            to.items.append(current) // 10.
            to.items.removeAll(where: { $0.name == nil}) // 11.
            from.items.removeAll(where: { $0.id == current.id }) // 12.
            if from.items.isEmpty {
                from.items.append(ScheduleItem(name: nil)) // 13.
            }
            listData[toIndex].items = to.items // 14.
            listData[fromIndex].items = from.items // 15.
        }
    }

    func dropUpdated(info: DropInfo) -> DropProposal? {
        return DropProposal(operation: .move) // 16.
    }

    func performDrop(info: DropInfo) -> Bool {
        self.current = nil // 17.
        return true
    }
}
{{< / highlight >}}

From the code above, there are some notable operations I do:
0. First we need to set the item of the current schedule object.
1. Then we need to bind the `listData` which is the whole `CalendarViewModel`.
2. This step we need to bind also the current `dragging` schedule item. Remember schedule item is part of `CalendarViewModel`.
3. Pretty obvious step, block any operation if both item and current are the same things.
4. Retrieving the correct `Calendar` model that contains the current dragged item.
5. Retrieving the correct `Calendar` model that contains the destination item.
6. Obvious step, both step numbers 4 and 5 should not be the same object. I make it as `var` because we will mutate the value of the struct.
7. We start to filter only take the destination item that matches with the `id` of the current schedule object from number 1.
8. Now we need to get the index of the calendar object which contains the dragged item.
9. We also need to get the index of the calendar object which contains the destination item.
10. Now we will add the dragged item to the destination array of items.
11. We also need to remove all of the items that contain `nil` name. This is one of the tricks I need to do in order for the item can be dragged to the empty calendar. (I will explain in the caveat section).
12. After that we need to remove the item from the source calendar.
13. This step, is part of the caveat section, where we need to append the source calendar an empty item object (fake object) if the array of items on that calendar is empty.
14. Now we need to assign the item of our **Bindable** calendar object. This one is assigning the destination object with the new list of items.
15. Same as step 14 but this one is assigning the source object with the new list of items.
16. One of the delegate functions is `dropUpdated` which we need to feed our `DropDelegate` with the `DropProposal`. `DropProposal` is the behavior of the drop action, which we need to declare the `DropOperation`. There are 4 types of `DropOperation` which are cancel, copy, forbidden, and move. See more info [here](https://developer.apple.com/documentation/swiftui/dropoperation). I would assume this one is for telling the system what is the expected behavior of the dropping action then the system can decide what the system can do for resolving the session of the drag and drop operation, however, I don't see any effect in our use case (yet), probably because we only move the text `UTType` instead of other possible `UTType`.
17. `performDrop` is the delegate function to determine whether we can perform the drop interaction or not. We set the return to `true`, but we set back the `current` or dragged item as `nil` to tell the system we finish dragging the item.

Above are the explanation of steps I made within my `DropDelegate` object. Now let's take a look back on step 11 with the **fake** object at the section below.

> To understand more about DropDelegate, DropInfo and DropProposal, please visit this documentation from Apple. https://developer.apple.com/documentation/swiftui/dropdelegate.

![calendar-gif](/img/calendar/calendarView.gif)

## Caveat

There is a single caveat here that I need to do in order we can do any drag and drop behavior within an item to another calendar that has no data/item in their calendar view. If we only drag the calendar it is possible to do so because we will apply all of the `onDrag` modifiers to all of the calendar views, then the system can tell the object that we want to compare and swap the calendar. But in this case, we want to drag the sub-item of the calendar, and the best case we should do in the UI part, that we will put an empty `Spacer` to the `Calendar` view instead of the `Item` view and we only put the `onDrag` behavior on the `Item` view. However, this is not possible, because we will make the `onDrag` and `onDrop` not recognizing the destination item view (because we only put a `Spacer` and we can not put the `onDrag` and `onDrop` in that spacer since no data can be compared within that spacer). So in order we can compare the item, we need to make a fake item object with a `nil` string name, and we place it all over the `Calendar` view that has no sub-items, then we can still put `onDrag` and `onDrop` modifier on top of that `Item` view, and the system can recognize the item to be compared. At the moment, I am still thinking this is a hacky way and not clean enough for this behavior, if anyone found out the better solution please [let me know!]((https://twitter.com/michaelabadiii)!).

## Conclusion

In my opinion, Drag and Drop behavior in SwiftUI are quite hard to implement at first and need a bit learning curve to understand how it behaves. However by the time you try it out, it is easier than I thought. Both of them indeed are quite tricky to implement, but it doesn't mean we need to scare of them. I suggest trying it out and you can make a lot of cool stuff with it. I hope my article could help you to understand more about the basic foundation of how to do drag and drop in SwiftUI, if you have any questions and better solutions please reach me out on [twitter](https://twitter.com/michaelabadiii)!

{{< sponsor2 >}}
