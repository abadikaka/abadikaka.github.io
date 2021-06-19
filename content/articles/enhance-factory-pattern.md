---
# Common-Defined params
title: "Enhance Factory Pattern capabilities for DI in Swift within 3 minutes"
date: "2021-02-11"
description: "Implementing Factory Pattern in Swift"
categories:
  - "iOS Development"
tags:
  - "Design Pattern"
  - "Swift"
lead: "Implementing Factory Pattern in Swift" # Lead text
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

Factory pattern is one of the common patterns in a programming language. It is one of the creational design patterns that provide a high level of flexibility for your code. One of the interesting use cases that could make Factory pattern become a handy tool to use is **Dependency Injection**. Normally whenever we want to inject an interface as a dependency on an object, we could easily pass it through the initializer or set it from the property. See the example below

{{< highlight swift "linenos=inline" >}}
class SampleViewModel {
  private var networkManager: NetworkProtocol
  private var databaseManager: DatabaseProtocol
  init(networkManager: NetworkProtocol, databaseManager: DatabaseProtocol) {
      self.networkManager = networkManager
      self.databaseManager = databaseManager
  }
}
{{< / highlight >}}

In the above example, we are trying to inject the **SampleViewModel** with dependencies of network and database manager (Remember, always depend on abstraction). It is the most simple way to do DI over initializer, and problems will start to occur once we have multiple dependencies and need to inject too many parameters. Of course, then we try the second approach like below.

{{< highlight swift "linenos=inline" >}}
class SampleViewModel {
  typealias Dependencies = HasNetwork & HasDatabase
  private var dependencies: Dependencies
  init(dependencies: Dependencies) {
      self.dependencies = dependencies
  }
}
{{< / highlight >}}

In the second approach, we simply pass a typealias with the rules if both of the dependencies are encapsulated into an object that conforms to both of them, which is called protocol composition. It is a very swifty way and looks so neat. There will be no problem until then we need to pass every single dependency to the initializer or we could end up with multiple & operator into our typealias. How about if we are using one object that could easily produce this for us? Yes! that’s the factory. See below example

{{< highlight swift "linenos=inline" >}}
class SampleViewModel {
  private var factory: ConfigurationFactory = ConfigurationFactory()
  init() {
      self.dependencies = factory.createDependenciesOne()
  }
}
{{< / highlight >}}

See Option 3, we could just have a factory instance, and then call out the function to create the dependencies for us. See the below picture for the big picture of our Factory class. It has the function of creating the dependencies for us, or either provide a function to create a singular object of some dependency.

Below is the complete code

{{< highlight swift "linenos=inline" >}}
protocol NetworkProtocol {}
protocol DatabaseProtocol {}  

protocol HasNetwork {
  var network: NetworkProtocol { get }  
}

protocol HasDatabase {
  var database: DatabaseProtocol { get }  
}  

final class ConfigurationFactory {
    func createNetworkManager() -> NetworkProtocol {
      return NetworkManager()
    }

    func createDatabaseManager() -> DatabaseProtocol {
      return databaseManager()
    }

    typealias DependenciesOne = HasNetwork & HasDatabase
    func createDependenciesOne() {
      return ObjectWithThatTwoDependencies()
    }
}
{{< / highlight >}}

## TIPS & TRICK

1. You can create a singleton factory in case you want to make one factory that can easily accessible by any object. However, bear in mind, in order to keep our **singleton** not become **an anti-pattern, as most people try to avoid it**, keep it **stateless**, only for creating a necessary object. By creating our singleton stateless and only for producing an object, then it’s fine, don’t need to worry about testing. Remember, the problem of a singleton in DI mostly, because of **singleton can produce a shared state** and **provide a stateful class**. So keep it clean.

2. Another trick we can create a **Container**, and this container can conforms to multiple **Factory**, then store this container into the top-level object of your application, or even make it a singleton if needed, and this **Container** is the only one assemble most of Factory protocol that could produce multiple objects that needed by its accessor.

3. Explore more about [Resolver](https://quickbirdstudios.com/blog/swift-dependency-injection-service-locators/) pattern. This pattern would be a great example of how Factory pattern can be a powerful tool for DI. [Swinject](https://github.com/Swinject/Swinject) is one example of how Resolver pattern works and how Factory pattern is working under the hood.

### Container Example

{{< highlight swift "linenos=inline" >}}
class DependenciesContainer: HasNetwork, HasDatabase {
  var database: DatabaseProtocol {
    return DatabaseManager(type: .coreData)
  }
  var networkManager: NetworkProtocol {
    return NetworkManager(urlSession: .shared) // or .inMemory
  }
}

extension DependencyContainer: ViewModelFactory {
  func makeSampleViewModel() -> SampleViewModel {
    return SampleViewModel(dependencies: self)
  }
}
{{< / highlight >}}

That’s all for today's knowledge about factory patterns. Hope it is very helpful in addition of one of the most powerful **Creational Design Pattern**. Stay tuned for the next article!
