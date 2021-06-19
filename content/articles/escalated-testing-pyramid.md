---
# Common-Defined params
title: "Escalated Testing Pyramid"
date: "2020-09-21"
description: "Understand Pyramid of Testing"
categories:
  - "iOS Development"
tags:
  - "Testing"

# menu: main # Optional, add page to a menu. Options: main, side, footer

# Theme-Defined params
# thumbnail: "img/wrapper.png" # Thumbnail image
lead: "Understand Pyramid of Testing" # Lead text
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

As a normal software engineer, we always remember the rule of thumb of testing. We always heard the word of the testing pyramid which consists of 3 things :
1. Unit Testing
2. Integration Testing
3. UI Testing

![pyramid](/img/test-pyramid.png)

**Unit Testing** is the foundation of your test suite that will be made up of unit tests. Your unit tests make sure that a certain unit (your subject under test or SUT) of your codebase works as intended. Unit tests have the narrowest scope of all the tests in your test suite. The number of unit tests in your test suite will largely outnumber any other type of test. Normally we test every single function of each class to be able to cover 100% test coverage which is impossible. Below is an example of UT in Swift

```
func testMultiplication() {
   let a = 4
   let b = 2
   XCTAssert(a * b, 8)
}
```

**Integration Testing** is another testing term for testing the integration or communication between each other component of the application such as database, networking, or another layer of an object that needs to integrate with each other. Here are a good explanation and example of Integration testing in swift by [John Sundel](https://www.swiftbysundell.com/articles/integration-tests-in-swift/).

**UI Testing** is the last layer which is the layer that facing the end-user directly, in this case, we normally call it as an end to end test. The most non-efficient way yet the most accurate is manual testing for UI Testing since it involves an interaction of human behavior which sometimes leads to the BDD testing concept. We will put our accessibility identifier for each UI Element to be able to detect the target component in automation.

---

Most of the time all of the basic three testings above are not enough, sometime the application is growing bigger and needs a more maintainable testing approach as well a more concise testing structure. Finally, most of the company try to embrace more testing and find more ways to make testing more precise. Escalation of Testing Pyramid consists of more structure. We can breakdown more into several structures. This is one of the enhanced testing pyramids that I found out. (Order start from the lowest point)

![pyramid](/img/test-pyramid-2.png)

- Unit Testing
- Snapshot Testing → Acceptance Test
- Integration Testing
- UI Local Testing → System Test
- UI Mock Server Testing → System Test
- E2E Testing → Production environment

What was the difference between the former pyramid? So this one we can see some enhancement such as **Snapshot Testing** and **UI Testing**. Besides that, some companies also put more additional tests like **Contract testing and API Testing** as well.

**Snapshot Testing**: Snapshot testing is a testing method that takes the precedent state of the view and compares it with the latest state of the view, if there are some changes, it will tell us if the test is broken because of the snapshot mismatch. Normally this kind of test also considers an acceptance test. The way it works, the system will store the latest state of the view to the memory then later on when someone runs the test, it will compare the newest value with the stored value. It compare either the view hierarchy or the state of the logic within the view itself. Snapshot testing useful to remind the developer if we need to double-check on a specific page if it may break some old logic or cause some issues on it. If there is no issue and all parties accept it, we need to update the test with the newest state of that view. We can use the famous FBSnapshotTesting for snapshot tests in mobile development.

```
import FBSnapshotTestCase
class FBSnapshotTestCaseSwiftTest: FBSnapshotTestCase {  
    override func setUp() {    
       super.setUp()    
       recordMode = false  
    }   
    func testExample() {    
       let view = UIView(frame: CGRect(x: 0, y: 0, width: 64, height: 64))    
       view.backgroundColor = UIColor.blue
       FBSnapshotVerifyView(view)     
       FBSnapshotVerifyLayer(view.layer)
     }
}
```

**UI Local Testing and UI Mock Server Testing**: This testing is also driven by the concept of BDD. Somehow we need to test UI behavior that may work with the domain model or the business logic. We may mock the object which is very useful rather than uses the real object for that test. It is called as **[TestDouble](https://martinfowler.com/bliki/TestDouble.html)** method which taking the Mock object to test the behavior of the flow instead of comparing the state of the real object at the end of the state (Old classic test look for the end state of the class, meanwhile mock test looking for the correct behavior flow of specific test with ignoring the end state within the state, as long as the behavior working perfectly). We can divide this state by using local object testing in our machine before we are using real mock objects from the server that can configurable by the QA. We may need a Docker setup and start our own server for mocking the response from the server and also mocking our domain model object into the test.

**E2E Testing**: This is the last testing which is UI Testing. This one is using production data and environment and normally will be tested with manual QA for better precision.
