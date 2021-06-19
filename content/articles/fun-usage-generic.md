---
# Common-Defined params
title: "Fun usage of Generic implementation in Swift"
date: "2020-10-31"
description: "Implementing Generic in Swift"
categories:
  - "iOS Development"
tags:
  - "Swift"
lead: "Implementing Generic in Swift" # Lead text
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

Most of us know that generic in programming means we able to do something not limited to one specific type but has more wide selection on the same basis. The most common examples are `Queue` or `Stack` use cases. Let say we would like to implement an implementation of both problems not limited to one type of data type, we would like to use a Generic way to solve this problem like the code below

{{< highlight swift "linenos=inline" >}}
class Stack<T> {
    private var data = [T]()

    var count: Int {
        return data.count
    }

    func push(_ item: T) {
        data.append(item)
    }    
    @discardableResult
    func pop() -> T? {
        if !data.isEmpty {
            return data.popLast()
        } else {
            return nil
        }
    }
}
{{< / highlight >}}

With the code above, the word `T` can be replaced with anything, and it will refer to the Generic type. Later on, we could use the implementation above like below

{{< highlight swift "linenos=inline" >}}
let stack = Stack<Int>()
stack.push(5) // add 5
stack.push(7) // add 7
stack.pop() // return and remove 7
{{< / highlight >}}

How about if we want to hide the internal function of this method and would like to only know the abstraction of the generic implementation itself by only understand the interface? That’s how the protocol associated type comes in. It is quite handy. [Protocols with associated types are one of the most powerful, expressive, and flexible features of Swift](https://www.hackingwithswift.com/articles/74/understanding-protocol-associated-types-and-their-constraints), but there is some complexity behind it and we may need to control their complexity by adding constraints using them, which restricts how they are used.

## USE CASE

Okay now, let’s go-to real use case implementation. Now let say we have a modular architecture, and we would like to have a Factory class that able to generate the correct view output to be present on the screen. Let say we have 3 classes that will work together on a screen, which is `Cart` , `Banner` , and `Recommendation` . These 3 classes will be implemented inside a big ViewController and work on different modules. The easiest way is to make an **Engine** class on each module to return the correct View to the main app. What is the hassle here? You need to create each Engine for each module! Why don’t we just have one **Generic** Engine class that able to regenerate each necessary module base on the input and output we need?
First, let’s define the protocol type of this generic :

{{< highlight swift "linenos=inline" >}}
protocol GenericEngine {
  associatedtype EngineType
  associatedtype Output
  func buildEngine() -> Output
}
{{< / highlight >}}

The above protocol will define the interface of our Generic engine which is accepting an Input and Output. Both of these are not being defined yet! That’s mean it is up to you how to define it explicitly later on the class implementation by using `typealias` or just make it implicitly base on the user input and let the compiler predict it base on the flow or constraint you add.

Finally, let’s create a Generic class base on this protocol. I combine some of the ways into one generic class.

{{< highlight swift "linenos=inline" >}}
final class GenericImplementation<T, V>: GenericEngine {
typealias EngineType = T
  typealias Output = V

  private lazy var factory: EngineFactory = EngineFactory<EngineType, Output>()

  func buildEngine() -> Output {
     let engine = factory.buildModule()
     let action = engine.action
     print("Got engine type :")
     action.printType()
     if let vc = engine.viewController as? Output {
        return vc
     } else {
        fatalError("Ask wrong output type detected")
     }
  }
}
{{< / highlight >}}

Look at the above class, I have defined a GenericImplementation class that conforms to a GenericEngine, however this Generic class accepting both T and V type inference. T in here is a generic type without any constraints, and base on the protocol of GenericEngine, it can be the EngineType, which means this EngineType can be anything, the same applies to the Output. We can actually explicitly tell what we want to refer with typealias there directly, however, if we declare it in this class, it won’t be a generic one as intended. The last one is the buildEngine implementation we need to make an implementation of a class that can accept the EngineType parameters and produces an Output. However, in the implementation above, we also can validate the output if it is the correct type or not, if not then we need to let the compiler check it during runtime. Let see the rest of the class such as the factory class below

{{< highlight swift "linenos=inline" >}}
final class EngineFactory<Input, Output> {
   func buildModule() -> (viewController: UIViewController, action: ModuleAction) {
      let moduleAction: ModuleAction
      if Input.self == Cart.self {
        moduleAction = Cart()
      } else if Input.self == Banner.self {
        moduleAction = Banner()
      } else if Input.self == Recommendation.self {
        moduleAction = Recommendation()
      } else {
        moduleAction = Recommendation()
      }
      return (moduleAction.viewController, moduleAction)
    }
}

protocol ModuleAction {
   var viewController: UIViewController { get }
   func printType()
}
extension ModuleAction {
   func printType() {
     print("This one is \(type(of: self))")
   }
}
class Cart: ModuleAction {
   var viewController: UIViewController {
     return CartVC()
  }
}
class Banner: ModuleAction {
   var viewController: UIViewController {
     return BannerVC()
   }
}
class Recommendation: ModuleAction {
   var viewController: UIViewController {
     return RecommendationVC()
   }
}
class CartVC: UIViewController { }
class BannerVC: UIViewController { }
class RecommendationVC: UIViewController { }
{{< / highlight >}}

So the above class is the implementation of our Factory class to produce the correct Module type and Output type. Now how do we use this generic class? See below implementation for the details

{{< highlight swift "linenos=inline" >}}
let generic: GenericImplementation<Cart, CartVC> = GenericImplementation()
let module = generic.buildEngine()
print("This vc type is \(type(of: module))")
{{< / highlight >}}

Run the code above and feel free to change your engine type and output type and you can see when it is going to be correct and when is it going to be the wrong one. Happy coding!
