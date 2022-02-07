---
# Common-Defined params
title: "VIPER adoption to SwiftUI"
date: "2021-08-29"
thumbnail: "img/viper-pattern/viper.001.jpeg"
description: "Learn How To Use VIPER in SwiftUI"
categories:
  - "iOS Development"
tags:
  - "Design Pattern"
  - "Swift"
  - "SwiftUI"
lead: "Learn How To Use VIPER in SwiftUI" # Lead text
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

**VIPER** is one of the hottest architectures back then because of its separation of layers that isolated between each other which is also debatable. Some argue that VIPER is over-engineer and an overkill architecture, some argue that VIPER causes too many boilerplates of code and some argue that VIPER is one of the best architecture for iOS Development. All of these opinions are valid, we definitely can use VIPER depends on how complex is our project and other undefined constraints. In this article, I would like to share my experience to migrate `SwiftUI` gradually to our VIPER codebase without breaking any major concept of VIPER.

{{< sponsor3 >}}

# Definition
What is VIPER? VIPER is a robust and scalable architectural design pattern which consists of 5 vital elements; **View**, **Interactor**, **Presenter**, **Entity** and **Router**. The intention of such separation is for conforming to all of the SOLID principle paradigms, hence every single element of our application needs to be separated to solely focus on its main intention.

- **View** : This one is the view object part that focuses on the place where the view is belong to. This object will have a reference to a Presenter to communicate the interaction to the Presenter.

{{< highlight swift "linenos=inline" >}}

final class View: UIViewController, ViewInput {
  var output: ViewOutput?

  override func viewDidLoad() {
    super.viewDidLoad()
    output?.viewDidLoad()
  }

  func handleCalendarButton() {
    output?.didClickOnCalendarView()
  }

  // MARK:- ViewInput

  func update(_ viewModel: ViewModel) {
    // update the UI
  }
}

{{< / highlight >}}

- **Interactor** : Interactor is part where the business logic is living in. This object will maintain and be the only source of truth for the incoming and outcoming data. Interactor is also the place where we can map the data into the form that Presenter needs, keep in mind this mapped data is not view mapped model but more to module entity model.

{{< highlight swift "linenos=inline" >}}

final class Interactor: InteractorInput {
  weak var output: InteractorOutput?

  // MARK: - InteractorInput

  func fetchData() {
    // some api call or other business logic stuff and map the data to the ModuleEntity

    output?.received(_ data: data) // this is module entity
  }
}

{{< / highlight >}}

- **Presenter** : This object is the place that becomes the connector between `View, Router, and Interactor` hence `Presenter` will hold references of all of them. The Presenter's task is to map the data received from `Interactor` into the form that can be used in the View level, hence it can be into multiple forms of view model depend on the necessity of the view. `Presenter` is also the object that received a user interaction response from the View, so it can send back the flow to the `Interactor` to query the necessary data base on the user interaction. Meanwhile, the Presenter is also able to decide when the module needs a Router to send the flow outside of the scope of this module or even another view within the same module after retrieving specific flow from the Interactor response.

{{< highlight swift "linenos=inline" >}}
final class Presenter: InteractorOutput, ViewOutput, InteractorOutput {
  var router: RouterInput?
  weak var view: ViewInput?
  var interactor: InteractorInput?

  private var viewModel: ViewViewModel = .defaultModel

  // MARK: - ViewOutput
  func viewDidLoad() {
    interactor?.fetchData()
  }

  // MARK: - InteractorOutput
  func received(_ data: ModuleEntity) {
    viewModel = data.map { ViewViewModel($0) }
    view?.update(viewModel)
  }
}
{{< / highlight >}}

- **Entity** : Entity here is a dummy domain model object which do not have internal business logic. This entity should be stateless and only has constant data that cover the module entity. Some people suggest to breakdown the entity into a more specific sub-domain model, but it is a preference. An example of an entity is shown below

{{< highlight swift "linenos=inline" >}}
struct ModuleEntity {
  let title: String
  let description: String
  let groups: [String]
  let imageUrl: String  
}
{{< / highlight >}}

- **Router** : Router is pretty clear base on the name, this object will be the only place for routing the current view to another view of another module or view within the same module

{{< highlight swift "linenos=inline" >}}
final class Router: RouterInput {
  var view: UIViewController?

  //.. some other code here ..//

  // MARK: - RouterInput

  func showChangeDate() {
      let dateModule = dateBuilder.build() // Usage of Builder pattern
      view?.navigationController?.pushViewController(dateModule.viewController, animated: true)
  }

  func popBack() {
      view?.navigationController?.popViewController(animated: true)
  }
}
{{< / highlight >}}

> NOTE: If you notice, I made a weak ref of the view on Presenter instead of the opposite, the reason because, by default, UIViewController has a strong reference to the UINavigationController then if the view disappears and is deallocated, ARC will erase everything starting with Presenter, hence presenter reference in the view must be strong.

{{< sponsor >}}

# Problem with SwiftUI

In **UIKit** world, every single module or domain we can easily attach VIPER template to it and communicate each layer easily since UIView / UIViewController is a class and it can have an inheritance just in case we want to subclass the view from another super view. The View Input (abstraction to communicate between presenter and the view), that we declare is an AnyObject protocol, so class is obviously the better choice here. Besides, `UIKit` has a complete view controller lifecycle which makes the job easier to act accordingly with the data layer. How about `SwiftUI` ? SwiftUI has a different concept with the UIKit lifecycle, especially how it renders the UI. In SwiftUI normally we will act according to a reactive changes base on the latest data changes, in this case we can use `@Environment(\.scenePhase)` for a window change, or `.onAppear / .onDisappear` for view lifecycle. Since the View is a struct type, it also can not has an inheritance of another view. Another factor is our ViewInput abstraction for receiving a view input from the Presenter is bounded to class type protocol, so non-class type of our View is not suitable to has a ViewInput conformance.

Another thing is since `View` is not `UIView` or `UIViewController` but an opaque type of `some View`, the router is not able to easily navigate like it uses to do with `UIKit`.

Base on this observation there are two things we need to resolve:
- Communication between Presenter and View, and
- Router to navigate each view/module

# Solution

## View and Data Communication
**Adapter** pattern is needed in this case for acting as the `ViewInput` and an ObservableObject, hence based on that two conditions, it is a class type object. To maintain **SwiftUI** integrity to having a reactive flow with the Binding concept to an ObservedObject / StateObject; depend on context (See this [article](https://www.donnywals.com/whats-the-difference-between-stateobject-and-observedobject/) for the differences), and also without breaking the concept of VIPER, an adapter would be a handy intermediary object between those two. Our SwiftUI View will have a binding to the Adapter (as an ObservableObject), and our VIPER Presenter will communicate to the Adapter (as a ViewInput). Besides that, we also have presenter reference to the Adapter, and keep the flow only through this Adapter.

{{< highlight swift "linenos=inline" >}}

final class ViewAdapter: ObservableObject, ViewInput {
    typealias Presenter = ViewOutput

    lazy var panelView: AnyView = {
        AnyView(SwiftUIView(observedModel: self))
    }()

    private let presenter: Presenter

    @Published var viewModel: ViewViewModel = .defaultModel

    init(presenter: Presenter) {
        self.presenter = presenter
    }

    // MARK: - Adapter

    var input: ViewInput {
        return self
    }

    var output: ViewOutput {
        return presenter
    }

    // MARK: - ViewInput
    func update(with viewModel: ViewViewModel) {
        self.viewModel = viewModel
    }

}

{{< / highlight >}}

One interesting thing here is the usage of type-erasure `AnyView`. We can not store a reference of optional `opaque` type, otherwise, compiler will complain `an 'opaque' type must specify only 'any' 'anyobject' protocols and/or a base class` as part of the current restriction of an `opaque` type, then the workaround for this one is storing a **type-erasure** of a `View` which is `AnyView`.
Take a look at the `UIHostingController` documentation below
`open class UIHostingController<Content> : UIViewController where Content : View`, which states that `UIHostingController` is a generic class that requires a View type, hence the type-erasure `AnyView` is the sole candidate for accepting any View within the `UIHostingController`.

## SwiftUIView

After we create the adapter, below is the `SwiftUI` View implementation. As you can see, we only need one `ObservedObject` object which is the Adapter that acts as both ViewInput and has reference of `ViewOutput` (Presenter).

{{< highlight swift "linenos=inline" >}}
struct SwiftUIView: View {

    @ObservedObject var observedModel: ViewAdapter

    var body: some View {
        HStack {
            VStack {
                Text(observedModel.viewModel.startDateTitle)
                    .foregroundColor(.gray)
                Text(observedModel.viewModel.startDate)
                    .foregroundColor(.blue)
                    .onTapGesture {
                        observedModel.output.didTapChangeDate()
                    }
            }
        }
        .onAppear {
            observedModel.output.viewDidAppear()
        }
        .onDisappear {
            observedModel.output.viewDidDisappear()
        }
    }

}
{{< / highlight >}}

## Router

Since normally `Router` will hold a reference of `UIView` or `UIViewController`, we don't have much trouble navigating the module. However, in `SwiftUI`, it is not possible to get the `UIView` or the `UIViewController` from a View object because the underlying layout engine is different. One of the first thing to do is change the property data type to be `UIHostingController<AnyView>`, hence this is why we do need a type-erasure of a `View` from the `Adapter`. After assigning an `UIHostingController`, we can get the `navigationController` from the `UIHostingController` and navigate easily like below example

{{< highlight swift "linenos=inline" >}}

// Use case : User click calendar icon and want to change the date

final class Router: RouterInput {
  var view: UIHostingController<AnyView>?

  //.. some other code here ..//

  // MARK: - RouterInput

  func showChangeDate() {
      let dateModule = dateBuilder.build() // Usage of Builder pattern
      view?.navigationController?.pushViewController(dateModule.viewController, animated: true)
  }

  func popBack() {
      view?.navigationController?.popViewController(animated: true)
  }
}

{{< / highlight >}}

# Conclusion

**VIPER** is definitely a no-brainer **SOLID** and one of the cleanest architecture, however keep in mind the usage of VIPER is also depends on your project scale. In my opinion for a small project or medium-scale project sometimes VIPER can be overkill, but you can consider VIPER if you are planning to make a long-term scalable project. Another takeaway from this article is even if it is possible to use VIPER into our **SwiftUI** application, but somewhat it needs couple of tricks and additional object (**Adapter**) to do so without breaking the VIPER concept. As a result, I wouldn't recommend use VIPER with SwiftUI, since SwiftUI has a concept of Binding with the reactive concept programming like Combine, I would recommend using [**TCA**](https://github.com/pointfreeco/swift-composable-architecture), **Redux** or even only **MVVM**. What do you think about VIPER in SwiftUI? Let me know on [twitter](https://twitter.com/michaelabadiii)!

{{< sponsor2 >}}
