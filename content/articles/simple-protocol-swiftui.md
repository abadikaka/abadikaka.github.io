---
# Common-Defined params
title: "Simple Protocol Oriented in SwiftUI in 5 minutes"
date: "2020-07-24"
description: "Learn how to use Protocol Oriented in SwiftUI"
categories:
  - "iOS Development"
tags:
  - "SwiftUI"
# menu: main # Optional, add page to a menu. Options: main, side, footer

# Theme-Defined params
# thumbnail: "img/wrapper.png" # Thumbnail image
lead: "Learn how to use Protocol Oriented in SwiftUI" # Lead text
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

SwiftUI is a really powerful language since it was born in 2019. In 2020 (WWDC) Apple did announce a lot of improvement on SwiftUI. However, there is rarely a topic discussed good practices on how to do we able to implement protocol-oriented programming in SwiftUI. There are a lot of articles out there however I guarantee to cover that in 5 minutes of your time. This article is suitable for those who are just starting to learn about iOS Programming !.

---

Our end goal is to be able to fetch the data (mock data) and map it into our List view in SwiftUI. The application is about showing a list of the football player names. Normally we able to put everything under one Struct view object. Now let us take a look 3 main layer we will discuss :
1. View Layer
2. Business Logic Layer
3. Entity Layer

We will start with the **Entity Layer**. In order to show the data to the view through the view model, we need a dummy model, that must conform to `Identifiable` protocol (mandatory from SwiftUI) in order to the view to be able to identify the difference for each row.

{{< highlight swift "linenos=inline" >}}
// The dummy model
struct Player: Identifiable {
    let id: Int
    let name: String
    let number: Int
}
{{< / highlight >}}

Now let’s move to **BL Layer**. Basic knowledge, normally we will just have a single View Model handle everything however how about if we want to limit the caller to only able to know the datasource and action that available from the View Model ?. We can use protocol-oriented programming in this case.

{{< highlight swift "linenos=inline" >}}
// The datasource
protocol ItemViewModelDatasource {
    var data: [Player] { get set }
}

// The action capabilities
protocol ItemViewModelAction {
    func fetchItem()
    func addPlayer(_ player: Player)
}
{{< / highlight >}}

Both protocols above are the interface of datasource and action for our view model later. Now after this, since we are building with SwiftUI, we do need a class that conforms to `ObservableObject` and have a `@Publish` property wrapper to the corresponding datasource.

{{< highlight swift "linenos=inline" >}}
// Base model protocol
protocol ListViewModel: ObservableObject {
    var action: ItemViewModelAction { get }
    var datasource: ItemViewModelDatasource { get set }
}
{{< / highlight >}}

Above protocol is the interface of `ListViewModel` (which is a protocol) and we encapsulate and restrict it only to able to get the action and datasource (also give a setter capabilities for setting the bind-ed object). Now let make a full implementation on the view model class.

{{< highlight swift "linenos=inline" >}}
// Real class implementation
class PlayerListViewModel: ListViewModel, ItemViewModelAction, ItemViewModelDatasource {

    // MARK: - Datasource
    @Published var data: [Player] = []

    private lazy var _datasource: ItemViewModelDatasource = {
        return self
    }()

    var datasource: ItemViewModelDatasource {
        get {
            return _datasource
        }
        set {
            _datasource = newValue
        }
    }

    // MARK: - Action
    var action: ItemViewModelAction {
        return self
    }

    func fetchItem() {
        data = [
            Player(id: 1, name: "G. Donnaruma", number: 1),
            Player(id: 2, name: "Andrea Conti", number: 12),
            Player(id: 3, name: "G. Bonaventura", number:5),
            Player(id: 1, name: "Zlatan Ibrahimovic", number: 21)
        ]
    }

    func addPlayer(_ player: Player) {
        data.append(player)
    }
}
{{< / highlight >}}

As you can see, the view model needs to conform to our `ObservedObject` protocol interface as well as the datasource and action interface. Don’t forget to put property wrapper `@Published` in front of the data property. This is a mandatory step since this wrapper functionality is to inform our view that this one is the data from the observable object wants to notify the listener.

{{< highlight swift "linenos=inline" >}}
struct ContentView<Model>: View where Model: ListViewModel {

    @ObservedObject var viewModel: Model

    @State var items: [String] = [
        "G. Donnarumma",
        "Andrea Conti",
        "Jack Bonaventura",
        "Zlatan Ibrahimovic"
    ]

    var body: some View {
        NavigationView {
            List {
                ForEach(items, id: \.self) { item in
                    Text("Item - \(item)")
                }
            }
            .navigationBarTitle("AC Milan Player")
        }
    }
}
{{< / highlight >}}

In the last step, we can create our simple view as above. When we declare the struct we need to put the generic `Model` type after the struct name, along with the supported type which is `ListViewModel`. This way we always guarantee that the model that is passed by the caller would always be a `ListViewModel` abstraction. In above code, we are still using local `@State` data which is items. Now we want to change it with the one from the view model through our abstraction. Here are the changes below :

{{< highlight swift "linenos=inline" >}}
struct ContentView<Model>: View where Model: ListViewModel {

    @ObservedObject var viewModel: Model

    @State var items: [String] = [
        "G. Donnarumma",
        "Andrea Conti",
        "Jack Bonaventura",
        "Zlatan Ibrahimovic"
    ]

    var body: some View {
        NavigationView {
            List {
                ForEach(items, id: \.self) { item in
                    Text("Item - \(item)")
                }
            }
            .navigationBarTitle("AC Milan Player")
        }
    }
}
{{< / highlight >}}

As you can see above, we are binding the List with data from the view model instead local state. Also we are triggering the query data from .onAppear method of the list to get the data. Once you get the data it will show the data correctly as shown on the picture below.

![swiftprotocol](/img/simple-protocol.png)

That’s all you need to make a good separation of view and bussiness logic in SwiftUI.
