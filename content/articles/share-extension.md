---
# Common-Defined params
title: "Easy way to do Share Extension in iOS"
date: "2020-04-24"
description: "Implementing Share Extension in Swift"
categories:
  - "iOS Development"
tags:
  - "Swift"
lead: "Implementing Share Extension in Swift" # Lead text
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

Wonder how to make your application can share any picture or file or text even URL into our own application? Well everyone knows we can use App Extension and create our extension for doing it. But imagine if we have multiple clients and need to use a specific token for each client, or we do have many configurations for the UI that can be displayed differently according to the clients, how we can achieve that? This tutorial will try to help you and make life easier!

First of all, there are 2 big pictures that we will do :
1. Configure our app into 3 different dynamic frameworks with one main application, one framework for selector UI, one for Sharing Storage, one for NetworkRequest and the main app is our own Share Extension UI.
2. Create an App Extension for the sharing methods.

Let start with the big picture from the picture above about what’s going on next. The share view controller will call out the CustomUIKit frameworks for showing the UI of the user (all of the Business logic of it will be covered inside that UI). Second, during that process in the setup logic, we are going to call and set the sharing access token that coming from the main application and use it into the Custom UI Kit for displaying the data. We also can use the token for requesting a network request into the NetworkFramework. This way we are trying to implement SOLID principle into our Sharing Extension !. Sharing View Extension will only have one small amount of logic only for routing what it is going to do with the object.

Let begin with put new target into an existing of any project that you have. Choose File -> New Target -> Share Extension

![share](/img/share-extension-1.png)

After we put a Share Extension, we must check whether the App Group identifier is matching with each other or not. App Group Identifier is the key to hold the connection between the container and the Extension to communicate with each other. Both of the Parent App and the Extension is having their own lifecycle, hence they can not easily send the data or communicate. Apple provides an App Group identifier to easily detect which Parent app that having matching extension through the apple container (Photos, Browser, Video, etc).

In the main project, see the Signing & Capabilities section, make an App Identifier (prefix with group and suffix is container by default)

![share](/img/share-extension-2.png)

In the share.entitlements also check whether the App Groups object has one of the same identifier of the targeted application
![share](/img/share-extension-3.png)

The next steps are we are going to make 3 different dynamic frameworks to encapsulate each project into different microservices.
1. SelectorUIViewKit : This framework holds the responsibility to display SelectionView that might be reusable on another project (Pure View). I will use SwiftUI.
2. StorageKit : This framework holds the responsibility for sharing a storage system that useful for doing any CRUD process in our database. It can be generic storage that can use any kind of database that we want to implement. For this tutorial, I will use Apple Native UserDefaults.
3. NetworkKit : This framework holds the responsibility to network processes throughout our projects. For this tutorial, I will use the default URLSession.

---

Start with the SelectorUIViewKit, create a new project, and let start with a new view, we can call it as SelectorView.

{{< highlight swift "linenos=inline" >}}
//
//  SelectorView.swift
//  SelectorUIViewKitTutorial
//
//  Created by Michael Abadi Santoso on 1/27/20.
//  Copyright © 2020 Michael Abadi Santoso. All rights reserved.
//
import SwiftUI

public struct BaseView: View {
    let user: [User]

    public var body: some View {
        SelectorView(user: user)
    }
}

private struct SelectorView: View {
    let user: [User]

    var body: some View {
        NavigationView {
            List {
                ForEach(user, id: \.id) { user in
                    VStack(alignment: .leading) {
                        Text("NAME: \(user.name)")
                        Text("JOB: \(user.job)").padding(.top, 10)
                    }
                }
            }.navigationBarTitle("User")
        }
    }
}

struct SelectorView_Previews: PreviewProvider {
    static var previews: some View {
        SelectorView(user: [User(id: "test", name: "Michael", job: "Programmer")])
    }
}
{{< / highlight >}}

This SelectorView is pretty straightforward, it contains ListView displaying a name and the job description. After that, we need to create a SelectorEngine. This class will provide the public capability for any project that wants to import the framework. Some of the capabilities I will expose to the public are presentSelectorView and getSelectorView.

{{< highlight swift "linenos=inline" >}}
//
//  SelectorEngine.swift
//  SelectorUIViewKitTutorial
//
//  Created by Michael Abadi Santoso on 1/27/20.
//  Copyright © 2020 Michael Abadi Santoso. All rights reserved.
//
import Foundation
import SwiftUI

/// User model
public struct User: Identifiable, Codable {
    public let id: String
    public let name: String
    public let job: String

    public init(id: String, name: String, job: String) {
        self.id = id
        self.name = name
        self.job = job
    }
}

/// Class to represent the engine of this framework.
public class SelectorEngine {

    public init() {}

    /// Presenting a selector view method.
    /// - Parameters:
    ///   - user: the user model.
    ///   - viewController: the source view controller to present the selector view.
    public func presentSelectorView(withUsers user: [User], from viewController: UIViewController) {
        let contentView = BaseView(user: user)
        let contentViewController = UIHostingController(rootView: contentView)
        let contentNavigationController = UINavigationController(rootViewController: contentViewController)
        viewController.present(contentNavigationController, animated: true, completion: nil)
    }

    /// Retrieving the base view of the selector view.
    /// - Parameter user: the user model.
    public func getSelectorView(withUsers user: [User]) -> BaseView {
        return BaseView(user: user)
    }
}
{{< / highlight >}}

1. presentSelectorView(withUsers user: [User], from viewController: UIViewController) : The engine will ask the project to just give a datasource of User model that defined by the framework and the source view controller. This way this framework purely loose coupled with the datasource and the view.

2. getSelectorView(withUsers user: [User])->BaseView : This function will return the base view as a SwiftUI View. I hide the specific SelectorView on this framework by the BaseView instead. This way, the framework won’t know anything about SelectorView.

---

Now let’s move to the interesting part which is StorageKit. This framework solely will focus on storing and sharing the data between each other. For your information, the concept of Share Extension is, it separately has its own lifecycle with the parent application. That way we can not store the data into the same database engine that just lives on one of the lifecycles. However, Apple has the bridging database that can be useful for sharing the data between Extension and parent application which is through UserDefaults with a suiteName. Suite name here is the app group identifier that identifies one extension is belong to the correct parent application.


{{< highlight swift "linenos=inline" >}}
//
//  StorageKit.swift
//  StorageKitTutorial
//
//  Created by Michael Abadi Santoso on 1/27/20.
//  Copyright © 2020 Michael Abadi Santoso. All rights reserved.
//
import Foundation

public let appGroupIdentifier = "group.com.michaelabadi.ShareKitTutorial.container"

/// StorageKit task is loading and saving any kind of token authentification
public class StorageKit {

    public enum StorageType {
        case `default`
        case sharing
    }

    private let type: StorageType

    public init(type: StorageType) {
        self.type = type
    }

    /// Function for saving the token.
    /// - Parameter token: the existing token.
    public func saveToken(_ token: String) {
        getUserDefault()?.set(token, forKey: "token")
    }

    /// Function for loading the token.
    public func loadToken() -> Any? {
        return getUserDefault()?.value(forKey: "token")
    }

    /// Function for delete the existing token.
    public func deleteToken() {
        getUserDefault()?.removeObject(forKey: "token")
    }

    private func getUserDefault() -> UserDefaults? {
        switch type {
        case .default:
            return UserDefaults.standard
        case .sharing:
            return UserDefaults(suiteName: appGroupIdentifier)
        }
    }

}
{{< / highlight >}}

The Storage Kit is easy enough to understand. There are basic concepts of database operation which is Create (Save), Read (Load), Update (Save) and Delete operation. UserDefaults(suiteName:) is the most important one for the Extension operation. However, you can expand this StorageKit to be able to use another kind of database system as you wish.

---

Next is **NetworkKit**. This is an extra Framework that I want to make not only for ShareExtension but for the parent application. You can reuse this for other purposes as well. However, you can have simple URLSession implementation inside the ShareKit instead. (This step is optional).

{{< highlight swift "linenos=inline" >}}
//
//  NetworkKit.swift
//  NetworkKitTutorial
//
//  Created by Michael Abadi Santoso on 1/27/20.
//  Copyright © 2020 Michael Abadi Santoso. All rights reserved.
//
import Foundation

public protocol NetworkBaseRequest {
    var endpoint: String { get }
    var params: [String: Any]? { get }
}

public typealias HttpCompletion = (_ result: Any?,_ error: Error?) -> Void

public enum HttpMethod {
    case get
    case post
}

public let mockResult = [
    ["id":"0", "name": "Michael", "job": "Programmer"],
    ["id":"1", "name": "Niramon", "job": "QA"]
]

/// Network class for sending any HTTP Request
public class NetworkKit {

    public init() {}

    /// Send an HTTP Request.
    /// - Parameters:
    ///   - request: The request object of detailed destination.
    ///   - type: type of the http request.
    ///   - completion: the result or error.
    public func sendRequest(_ request: NetworkBaseRequest, type: HttpMethod, completion: HttpCompletion) {
        completion(mockResult, nil)
    }

    /// Setup the network with proper token.
    /// - Parameter token: The token needed for authentification of the networking.
    public func setupAuthentification(withToken token: String) {
        // You can setup your token as header or whatever it is in here as well
    }
}
{{< / highlight >}}

1. sendRequest() : This function is for getting the result that we are going to use for mapping the data into the model before displaying it into the View.

2. setupAuthentification(withToken) : This function is for setting your own token as a request header or anything you want to do with the token on the Network setting.

Finally let see on our ShareKit Extension file which is ShareViewController.

{{< highlight swift "linenos=inline" >}}
//
//  ShareViewController.swift
//  SharingExtension
//
//  Created by Michael Abadi Santoso on 1/28/20.
//  Copyright © 2020 Michael Abadi Santoso. All rights reserved.
//
import UIKit
import StorageKitTutorial
import NetworkKitTutorial
import SwiftUI

/// Mocking Request class for the networking call
final class Request: NetworkBaseRequest {
    var params: [String : Any]? = nil

    var endpoint: String {
        return "v1/api/request"
    }
}

/// Share Extention main view
final class ShareViewController: UIViewController {

    private let storage = StorageKit(type: .sharing)
    private let network = NetworkKit()

    override func viewWillAppear(_ animated: Bool) {
        super.viewWillAppear(animated)
        setupToken()
    }

    /// Initialize the token from the main app and the extention take the token from main app
    private func setupToken() {
        if let token = storage.loadToken() as? String {
            network.setupAuthentification(withToken: token)
            print("TOKEN : \(token)")
        }
    }

    @IBSegueAction func addView(_ coder: NSCoder) -> UIViewController? {
        return UIHostingController(coder: coder, rootView: ShareContentView())
    }

}
{{< / highlight >}}

As you can see in the class interface above, the system will call setupToken() whenever the view interface appears to set up the shared token for reusing it. However, you must open the parent app at least once for saving the initial token before loading it in here. Also, notice on the top, we import all the necessary framework for doing all the operation we need. Don’t forget also to embed those frameworks beforehand on the main project. Now let run the application and see how it runs. (Don’t forget to set up the main project view and save the initial token on the main project as well (i.e. in the AppDelegate).

---

Okay, that’s all the easy way to develop a Share Extension as well the knowledge of how to utilize the dynamic frameworks into our application !. There is a lot of other x-factors that we need to understand more which is how to configure the subquery or activation rules for the Extension with their Info.plist, so we able to limit what kind of stuff we can share into our parent application, and also how do we take and process the item through NSExtensionItem. For instance, if you would like to put more restrictions on how many photos we can share, we must put the new rule on Extensions Info.plist file

![share](/img/share-extension-4.png)

Above Info.plist I put a limitation on the Maximum number of Photos I can share (20). Full list of rules can be shown in here: [Rules](https://developer.apple.com/library/archive/documentation/General/Reference/InfoPlistKeyReference/Articles/AppExtensionKeys.html)

So far, we have learned about Extensions and Dynamic framework's role in our application. Let me know if you have another question!
Full source code: [Project](https://github.com/abadikaka/ShareKitTutorial)
