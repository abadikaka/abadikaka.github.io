---
# Common-Defined params
title: "Utilize Combine for Search Engine in Swift"
date: "2021-01-24"
description: "Implementing Combine for Search Case in Swift"
categories:
  - "iOS Development"
tags:
  - "Combine"
  - "SwiftUI"
lead: "Implementing Combine for Search Case in Swift" # Lead text
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

**Combine**, is a new framework introduced by Apple since SwiftUI 1.0 back then in 2019. The architecture behind the scene itself is adopting the system of **FRP (Functional Reactive Programming)** which takes computation as their main computation of process combine with the propagation of data through the stream of the function. In this tutorial, I would like to explain how Combine can make your job done quicker than ever compare with the conventional way of code. Multiple-use cases of **Combine** usage have been explored such as Networking and any asynchronous process, but there are few of them explaining how we can utilize it for a property, especially an Observable property with a special `@Published` property wrapper. One of them is how do we do search operations easier with Combine. Let take a look at the old implementation below.

1. The SearchView need a completion and we need to send the value back from `onChange` method of the TextField
2. From the main view, we need to call `updateSearch` of the `viewmodel` in order to filter the data based on the updated string
3. It’s so inconvenient to do this every time we have a search bar

{{< highlight swift "linenos=inline" >}}
final class ViewModel: ObservableObject {
  @Published var searchTerm: String = ""
  private var cancellables = Set<AnyCancellable>()

  func updateSearch(text: String) {
    // Do data filtering
  }
}

struct SearchView: View {

    @Binding var searchTerm: String

    var onCompletion: (_ text: String) -> Void

    var body: some View {
        HStack {
            Spacer()
            Image(systemName: "magnifyingglass")

            TextField("Search", text: self.$searchTerm)
                .foregroundColor(Color.primary)
                .padding(10)
                .onChange(of: searchTerm, perform: { value in
                    onCompletion(value)
                })
            Spacer()
        }.foregroundColor(.secondary)
            .background(Color(.secondarySystemBackground))
            .cornerRadius(10)
            .padding(10)
    }
}

{{< / highlight >}}

With Combine, now we don’t need to have the completion handler on the SearchView and we can directly use.sinkoperator on the Published property. Look at the below implementation. What we can do is only do the binding to the property. The view can directly inject the property, and don’t need to do any completion handler anymore (we can remove it from the View).

1. Do `.receive(on: RunLoop.main)` for receiving the operation on the main queue so the UI will not get blocked. However, you can remove this since we never use any other thread when doing the search
2. `.sink` method will emit an output of the string whenever the string has a new value from the search operation.
3. Tips : You can debounce your operation in Combine and put it in any other thread, beware do the number 1 above if you are doing that

{{< highlight swift "linenos=inline" >}}
final class ViewModel: ObservableObject {
  @Published var searchTerm: String = ""
  private var cancellables = Set<AnyCancellable>()

  init() {
    bindingStock()
  }

  func bindingStock() {
      $searchTerm
          .receive(on: RunLoop.main)
          .sink(receiveValue: { [weak self] str in
              // Do filtering data here
          })
          .store(in: &cancellables)
  }
}

struct SearchView: View {

    @Binding var searchTerm: String

    var body: some View {
        HStack {
            Spacer()
            Image(systemName: "magnifyingglass")

            TextField("Search", text: self.$searchTerm)
                .foregroundColor(Color.primary)
                .padding(10)
            Spacer()
        }.foregroundColor(.secondary)
            .background(Color(.secondarySystemBackground))
            .cornerRadius(10)
            .padding(10)
    }
}
{{< / highlight >}}

Pretty neat right? We can achieve the search operation quicker than before with a more simplified syntax. Have fun and keep learning!
