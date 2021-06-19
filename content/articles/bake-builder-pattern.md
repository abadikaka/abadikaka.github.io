---
# Common-Defined params
title: "Bake a Builder Pattern in iOS"
date: "2020-05-13"
description: "Implementing Builder Pattern in Swift"
categories:
  - "iOS Development"
tags:
  - "Design Pattern"
  - "Swift"
lead: "Implementing Builder Pattern in Swift" # Lead text
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

Have you ever have a problem when designing a system that has a similar foundation and capabilities however the ingredients to form the object would vary? We may end up with an easy solution by providing different parameters for each subclass. However, multiple subclasses that need to override the constructor might also have to own those unnecessary properties into its interface. There is one design pattern that also considered as Creational Design Pattern which Android Design pattern by default having this pattern. It called as Builder Pattern.

> Q: How do Builder work in iOS?

Of course, there are many ways to make it work on iOS. Let understanding the basic with what is **Builder Pattern**. **Builder** is a creational design pattern that lets you construct objects step by step per user requirement. The pattern allows you to produce different types and representations of an object using the same construction code. It refers to a declarative way to construct an object. Builder pattern can be included as Reactive Programming as well!

---

Let start with the common problem, “A Car Company”. Company “A” is a big brand company and it has multiple products to be made. This company will make a “CarA” until “CarE”. They do have 5 types of cars, start from a car with a single door, sports car, family car, SUV, and a truck. We can achieve this easily by extend a “Car” base class and has the subclasses to override the necessary function, but we will end up with tons of subclasses and, and new parameters and more and more. We can simplify it by having one Base Class that owns a huge constructor, however, those subclasses need to override the constructor with all of the parameters from possible combinations. The constructor with lots of parameters has its downside, not all the parameters are needed at all times.

![Builder](/img/builder-pattern.png)

## SOLUTION: BUILDER PATTERN (THE CAR CONCEPT)

Yep, by using a builder pattern we eliminate that construction logic. The pattern will extract and define the behavior of construction into steps. We may have buildDoors, buildWheels, buildEngine, buildColor, buildExhaust. We don’t need to call every step if we don’t need it. If we do need to have another builder for another type of car with a different implementation of each step, we can make different builders and implement it in a different way!. For example, imagine a builder that builds a car from 4 sports wheel and 2 doors, a second one that builds everything with 4 normal wheel and 4 doors and a third one that uses 2 doors and a big wheel. By calling the same set of steps, you get a sports car from the first builder, a normal family car from the second, and a Sport SUV from the third. However, this would only work if the client code that calls the building steps is able to interact with builders using a common interface. We also able to escalate this by having another Manager class to manage the specific builder of the process of creating the car in the company

How do we achieve it in iOS? There are multiple ways, we can construct in the form of a block, or we can achieve it by setting it within the builder constructor, or even calling it in a declarative way. Let see how it differs from each other?

> NOTE: All of the code below is not in full code and not executable because I cut some of the unnecessary object. I try to show the core of the definition base on the use case. You can generate, modify or enhance this code to be more powerful as long as it is still conform to Builder Pattern definition.

### Declarative / by Function

{{< highlight swift "linenos=inline" >}}
// This one is a SportCarBuilder for building a sport car
// We just need two properties of wheel and color that can be added and modified
// However for engine and door has been defined by SportCar class
final class SportCarBuilder {

    private var wheel: Wheel = Wheel(type: .racing(tyre20))
    private var color: Color = Color(.red)

    func withColor(color: UIColor) {
      self.color = Color(color)
    }

    func withWheel(type: WheelType) {
      self.wheel = Wheel(type: type)
    }

    func build() -> SportCar {
        return SportCar(wheel: wheel, color: color)
    }
}

// This one is a SportCar class that subclass of the Car base class
// Car base class has 4 properties, this class will initialize through the base class with designated parameters that we define for SportCar
final class SportCar: Car {
  init(wheel: Wheel, color: Color) {
    super.init(doors: Door(2), engine: Engine(V85000), wheel: wheel, color: color)
  }
}

// How to call :
let sportCar = SportCarBuilder()
                .withColor(color: .red)
                .withWheel(type: .normal)
                .build()

// sportCar.wheel === .normal --> Defined by user
// sportCar.engine === Engine(V85000) --> Defined by SportCar implementation
{{< / highlight >}}

In the above code, it is pretty clear that when we want to get the sportCar Object, we can easily call the builder and add the necessary requirement/modifier into the builder. This way we are defining the behavior of what we want to the builder (declarative) without intervening with the implementation. SportCar also hides the Engine and Door properties from the Car base class because it will automatically be adjusted in the SportCar class instead.

### Block

{{< highlight swift "linenos=inline" >}}
// We reuse the above SportCarBuilder, we just change some interface
// Notice we instead are using the block for update the necessary properties
class SportCar {
  typealias Builder = (builder: SportCarBuilder) -> Void

  private init(builder: SportCarBuilder) {
    super.init(doors: Door(2), engine: Engine(V85000), wheel: builder.wheel, color: builder.color)
  }

  static func make(with builderBlock: Builder) -> SportCar {
    let builder = SportCarBuilder()
    builderBlock(builder)
    return SportCar(builder: builder)
  }
}

// This is the way we will use by defining the properties inside the block
let sportCar = SportCar.make { builder in
  builder.color = .red
  builder.wheel = .racing
}
{{< / highlight >}}

In the above code it is pretty straight forward, we will define the behavior we want through the completion block when we call the make function inside SportCar class. In most cases, the second way is used by Objective-C developer.

### Constructor Injection

{{< highlight swift "linenos=inline" >}}
// This is the base Car class that has all of the foundation materials to make a Car
class Car {
  let doors: Door
  let wheel: Wheel
  let engine: Engine
  let color: Color

  func buildCar() -> Car
}

// This is the subclass of Car which is a SportCar that has a private initializer
// Only bake the Car with the builder using a static function, however you can omit the static one and immediate using the constructor as well (your preference)
final class SportCar: Car {
  private init(builder: SportCarBuilder) {
    let door = Door(2)
    let engine = Engine(V85000)
    let wheel = builder.wheel
    let color = builder.color
    super.init(doors: door, engine: engine, wheel: wheel, color: color)
  }

  static func make(with builder: SportCarBuilder) -> SportCar {
    return SportCar(builder: builder)
  }
}

// This is the SportCarBuilder that just need color and wheel because the engine and the door will be automatically adjusted on the class who will use this by default
struct SportCarBuilder {
  let color: Color
  let wheel: Wheel
}

// This is the factory class for creating the necessary car on the company
// User will just need this class to retrieve the final product they need
final class CarFactory {

  func sportCar(with builder: SportCarBuilder) -> SportCar {
    return SportCar.make(builder: builder)
  }

  func truckCar(with builder: TruckCarBuilder) -> TruckCar {
    return TruckCar.make(builder: builder)
  }

  func SUVCar(with builder: SUVCarBuilder) -> SUVCar {
    return SUVCar.make(builder: builder)
  }

  func familyCar(with builder: FamilyCarBuilder) -> FamilyCar {
    return FamilyCar.make(builder: builder)
  }
}
{{< / highlight >}}

In the code above we can see that we do have CarFactory as a factory object that generates each car base on the Builder that the user wants to inject into the parameters. The user creates the necessary builder through their required constructor on the specific builder. SportCarBuilder doesn’t have engine properties because in the SportCar object that consumes the builder will automatically adjust the engine with the required engine only for a sport car. This way the specific requirement has been handled properly by the SportCar class.

---

Finally the conclusion, the goal of the builder pattern is to reduce the need to keep mutable state — resulting in objects that are simpler and generally more predictable. By enabling objects to become stateless, they are usually much easier to test and debug — since their logic consists only of “pure” input & output. There are other multiple pros to use the builder pattern, even though this is not iOS Common design pattern but we can achieve it. Remember, there is also some stuff to be considered before we go with the builder pattern. I will let you guys decide whether it is worth it or not for using this pattern on your project.
