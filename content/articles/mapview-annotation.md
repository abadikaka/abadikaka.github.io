---
# Common-Defined params
title: "MapView Annotation in SwiftUI"
date: "2020-09-11"
description: "Implementing MapView in SwiftUI"
categories:
  - "iOS Development"
tags:
  - "SwiftUI"
# menu: main # Optional, add page to a menu. Options: main, side, footer

# Theme-Defined params
# thumbnail: "img/wrapper.png" # Thumbnail image
lead: "Implementing MapView in SwiftUI" # Lead text
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

SwiftUI is very powerful for building an interactive UI at a fast pace. However, there are a couple of limitations still there such as some native API from UIKit like MKMapView from MapKit or search bar and other UIKit API. I will provide a tutorial for making a MapView in SwiftUI by using UIViewRepresentable as well as putting callback to the SwiftUI if we have clicked the annotation.

Some quick knowledge about several items below:

- **UIViewRepresentable** : A wrapper for a UIKit view that you use to integrate that view into your SwiftUI view hierarchy

- **Coordinator** : A SwiftUI view that represents a UIKit view controller can define a Coordinator type that SwiftUI manages and provides as part of the representable view’s context

- **MapKit** : UIKit API for Map behavior such as MKMapView and Annotation View and other native Map behavior

First of all, we need to make our Model for displaying the item inside the map. The model we can put title and it’s coordinate (latitude and longitude).

## CODE

{{< highlight swift "linenos=inline" >}}
final class Checkpoint: NSObject, MKAnnotation {
    let title: String?
    let countryCode: String?
    let coordinate: CLLocationCoordinate2D

    init(title: String?, countryCode: String?, coordinate: CLLocationCoordinate2D) {
        self.title = title
        self.countryCode = countryCode
        self.coordinate = coordinate
    }
}
{{< / highlight >}}

In the code above, the Checkpoint class is a class that represent our map point in the view. Once this model has been created let’s move to the creation of MapView

Create a new struct of **MapView** and make it conform to **UIViewRepresentable**.

{{< highlight swift "linenos=inline" >}}
import UIKit
import MapKit
import SwiftUI

struct MapView: UIViewRepresentable {

    // 1.
    var annotationOnTap: (_ title: String) -> Void

    // 2.   
    @Binding var checkpoints: [Checkpoint]

    /// 3. Used internally to maintain a reference to a MKMapView
    /// instance when the view is recreated.
    let key: String

    private static var mapViewStore = [String : MKMapView]()

    // 4.
    func makeUIView(context: Context) -> MKMapView {
        if let mapView = MapView.mapViewStore[key] {
            mapView.delegate = context.coordinator
            return mapView
        }
        let mapView = MKMapView(frame: .zero)
        mapView.delegate = context.coordinator
        MapView.mapViewStore[key] = mapView
        return mapView
    }

    // 5.
    func updateUIView(_ uiView: MKMapView, context: Context) {
        uiView.addAnnotations(checkpoints)
    }

    // 6.
    func makeCoordinator() -> MapCoordinator {
        MapCoordinator(self)
    }
}
{{< / highlight >}}

1. **AnnotationOnTap** is a completion to notify SwiftUI if we have clicked an annotation from MKMapView

2. **@Binding** is a property wrapper for checkpoints model that we need for this MapView for displaying each dot of the location

3. This key is for storing a single MKMapView instance in the memory. Using mapViewStore for handling if there is an existing instance of MKMapView on this particular screen. Why do we need this? There is a bug on MKMapView (UIKit) if we move to another screen and SwiftUI rerendering the struct of the SwiftUI View that contains this MapView it will create new MapView instead of reusing it while the old one still on the memory. It causes some bottleneck on rendering UI part for both SwiftUI and UIKit on the same point and it can cause a crash after several times.

4. This one overriding function from UIViewRepresentable to return the expected view

5. This one overriding function from UIViewRepresentable to attach a new view or do some additional layouting. In this case, we add the checkpoint to each annotation

6. This one also overriding function form UIViewRepresentable for coordinator which for mapping the delegation logic on MKMapViewDelegate

Okay once that view has been set up, now we can make the logic for notifying back to SwiftUI. We can not apply delegate in SwiftUI, so there is a Coordinator to put the business logic layer of pure Swift logic. Let make the MapCoordinator class.

{{< highlight swift "linenos=inline" >}}
final class MapCoordinator: NSObject, MKMapViewDelegate {    // 1.
    var parent: MapView

    init(_ parent: MapView) {
        self.parent = parent
    }

    deinit {
        print("deinit: MapCoordinator")
    }    // 2.    
    func mapView(_ mapView: MKMapView, didSelect view: MKAnnotationView) {
        view.canShowCallout = true

        let btn = UIButton(type: .detailDisclosure)
        view.rightCalloutAccessoryView = btn
    }

    // 3.    
    func mapView(_ mapView: MKMapView, annotationView view: MKAnnotationView, calloutAccessoryControlTapped control: UIControl) {
        guard let capital = view.annotation as? Checkpoint, let placeName = capital.title else { return }
        parent.annotationOnTap(placeName)
    }

}
{{< / highlight >}}

1. We need a reference to the MKMapView here for the coordinator able to return back the handler/logic we attach on it
2. This one is the delegate function from MKMapView (Put MKMapViewDelegate on this class as well) for displaying the rightCalloutAccesoryView
3. This one is for telling if we click on the accessory control and we will return back the placeName through the handler on the MapView

Once this has been set up now we can easily use the MapView on our SwiftUI.

{{< highlight swift "linenos=inline" >}}
struct SearchView: View {

    @ObservedObject var viewModel: SearchViewModel = SearchViewModel()    

    var body: some View {
        VStack {
                MapView(annotationOnTap: { title in
                    print("Title clicked", title)
                }, checkpoints: $viewModel.checkpoints, key: "SearchView")
                    .frame(height: UIScreen.main.bounds.height)
                    .offset(x: 0, y: 350)
            }
    }
}
{{< / highlight >}}

As you can see above we just need to pass the checkpoints model and key (can be anything) and viola, we can get the MapView working.
