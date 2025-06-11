- New Design System
- Container and adaptability
- The menu bar
- Architectural improvement
- General enhancements

## New Design System
- New background extension view
	- https://developer.apple.com/documentation/uikit/uibackgroundextensionview/
```swift
@MainActor
class UIBackgroundExtensionView
```
![[images/backgroundextensionview.png]]

- New glass material for custom components `UIGlassEffect`
  - https://developer.apple.com/documentation/uikit/uiglasseffect/
  - https://developer.apple.com/documentation/uikit/uiglasscontainereffect

## Containers and adaptivity
- UISplitViewController get support for inspector view
	- https://developer.apple.com/documentation/uikit/uisplitviewcontroller/column/inspector
- ![[images/Screenshot 2025-06-11 at 11.00.24 PM.png]]
## The menu bar
- Swipes from top reveals the menu
- ![[images/Screenshot 2025-06-11 at 11.03.22 PM.png]]
- ![[images/Screenshot 2025-06-11 at 11.04.07 PM.png]]![[images/Screenshot 2025-06-11 at 11.04.18 PM.png]]![[images/Screenshot 2025-06-11 at 11.04.40 PM.png]]
- Deffered menu item using responder chain
```swift
// AppDelegate.swift

extension UIDeferredMenuElement.Identifier {
    static let browserHistory: Self = .init(rawValue: "com.example.deferred-element.history")
}

// Create a focus-based deferred element that will display browser history
let historyDeferredElement = UIDeferredMenuElement.usingFocus(
    identifier: .browserHistory,
    shouldCacheItems: false
)

// Insert it into the app’s custom History menu when building the main menu
builder.insertElements([historyDeferredElement], atEndOfMenu: .history)
```

```swift

class BrowserViewController: UIViewController {

    // ...
  
    override func provider(
        for deferredElement: UIDeferredMenuElement
    ) -> UIDeferredMenuElement.Provider? {
        if deferredElement.identifier == .browserHistory {
            return UIDeferredMenuElement.Provider { completion in
                let browserHistoryMenuElements = profile.browserHistoryElements()
                completion(browserHistoryMenuElements)
            }
        }
        return nil
    }
}
```

![[images/Screenshot 2025-06-11 at 11.06.59 PM.png]]
## Architectural improvements
### Automatic observation tracking
- Using observable object and automatic observation tracking
```swift
@Observable class UnreadMessagesModel {
    var showStatus: Bool
    var statusText: String
}

class MessageListViewController: UIViewController {
    var unreadMessagesModel: UnreadMessagesModel

    var statusLabel: UILabel
    
    override func viewWillLayoutSubviews() {
        super.viewWillLayoutSubviews()

        statusLabel.alpha = unreadMessagesModel.showStatus ? 1.0 : 0.0
        statusLabel.text = unreadMessagesModel.statusText
    }
}
```
- Configuring a UICollectionView cell with automatic observation tracking
```swift
@Observable class ListItemModel {
    var icon: UIImage
    var title: String
    var subtitle: String
}

func collectionView(
    _ collectionView: UICollectionView, cellForItemAt indexPath: IndexPath
) -> UICollectionViewCell {
    let cell = collectionView.dequeueReusableCell(withReuseIdentifier: "Cell", for: indexPath)
    let listItemModel = listItemModel(for: indexPath)
    cell.configurationUpdateHandler = { cell, state in
        var content = UIListContentConfiguration.subtitleCell()
        content.image = listItemModel.icon
        content.text = listItemModel.title
        content.secondaryText = listItemModel.subtitle
        cell.contentConfiguration = content
    }
    return cell
}
```
- Using automatic observation tracking and updateProperties()
	- https://developer.apple.com/documentation/uikit/uiview/updateproperties()/
```swift
@Observable class BadgeModel {
   var badgeCount: Int?
}

class MyViewController: UIViewController {
   var model: BadgeModel
   let folderButton: UIBarButtonItem

    override func updateProperties() {
        super.updateProperties()

        if let badgeCount = model.badgeCount {
            folderButton.badge = .count(badgeCount)
        } else {
            folderButton.badge = nil
        }
   }
}
```
![[images/Screenshot 2025-06-11 at 11.10.18 PM.png]]
## Improvements to animations
- New animation option to auto animate on observation change
	- https://developer.apple.com/documentation/uikit/uiviewpropertyanimator/flushupdates/
	![[images/Screenshot 2025-06-11 at 11.11.35 PM.png]]

- Using the flushUpdates animation option to automatically animate updates
```swift
UIView.animate(options: .flushUpdates) {
    model.badgeColor = .red
}
```
- Automatically animate changes to Auto Layout constraints
```swift
UIView.animate(options: .flushUpdates) {
    // Change the constant of a NSLayoutConstraint
    topSpacingConstraint.constant = 20
    
    // Change which constraints are active
    leadingEdgeConstraint.isActive = false
    trailingEdgeConstraint.isActive = true
}
```
### Scene Updates
- Support of SwiftUI scenes in UIKit app using `UIHostingSceneDelegate`
- Setting up a UIHostingSceneDelegate
```swift
import UIKit
import SwiftUI

class ZenGardenSceneDelegate: UIResponder, UIHostingSceneDelegate {
    static var rootScene: some Scene {
        WindowGroup(id: "zengarden") {
            ZenGardenView()
        }

        #if os(visionOS)
        ImmersiveSpace(id: "zengardenspace") {
            ZenGardenSpace()
        }
        .immersionStyle(selection: .constant(.full),
                        in: .mixed, .progressive, .full)
        #endif 
    }
}
```
- Using a UIHostingSceneDelegate 
```swift
func application(_ application: UIApplication,
    configurationForConnecting connectingSceneSession: UISceneSession,
    options: UIScene.ConnectionOptions) -> UISceneConfiguration {

    let configuration = UISceneConfiguration(name: "Zen Garden Scene",
                                             sessionRole: connectingSceneSession.role)

    configuration.delegateClass = ZenGardenSceneDelegate.self
    return configuration
}
```
- Or if you want to programmatically launch a SwiftUI scene
```swift
func openZenGardenSpace() {
    let request = UISceneSessionActivationRequest(
        hostingDelegateClass: ZenGardenSceneDelegate.self,
        id: “zengardenspace")!
  
    UIApplication.shared.activateSceneSession(for: request)
}
```

## General enhancments
- HDR Color support
	- https://developer.apple.com/documentation/uikit/uicolor/linearexposure
```swift
// Create an HDR red relative to a 2.5x peak white
let hdrRed = UIColor(red: 1.0, green: 0.0, blue: 0.0, alpha: 1.0, linearExposure: 2.5)
```
- HDR Color PIcker
```swift
// maximum peak white of 2x
colorPickerController.maximumLinearExposure = 2.0
```
![[images/Screenshot 2025-06-11 at 11.22.52 PM.png]]
- New trait collection to move from HDR to SDR when the view is not in focus using new trait collection property
	- https://developer.apple.com/documentation/uikit/uitraitcollection/hdrheadroomusagelimit/
```swift
registerForTraitChanges([UITraitHDRHeadroomUsageLimit.self]) { traitEnvironment, previousTraitCollection in
    let currentHeadroomLimit = traitEnvironment.traitCollection.hdrHeadroomUsageLimit
    // Update HDR usage based on currentHeadroomLimit’s value
}
```
- NotificationCenter messages are now strogly types no more objects 
```swift
override func viewDidLoad() {
    super.viewDidLoad()

    let keyboardObserver = NotificationCenter.default.addObserver(
        of: UIScreen.self
        for: .keyboardWillShow
    ) { message in
        UIView.animate(
            withDuration: message.animationDuration, delay: 0, options: .flushUpdates
        ) {
            // Use message.endFrame to animate the layout of views with the keyboard
            let keyboardOverlap = view.bounds.maxY - message.endFrame.minY
            bottomConstraint.constant = keyboardOverlap
        }
    }
}
```

- UIApplicationLaunchOptionKeys are being deprecated
- Only init with window scene remains
	- Every other init deprecated
- Open URL now supports file URL and opens them in default app if it fails than it fallbacks to your app
- SF Symbol have new animations
	- https://developer.apple.com/documentation/symbols
```swift
var configuration = UIButton.Configuration.plain()
configuration.symbolContentTransition = UISymbolContentTransition(.replace)
...
configuration.symbolContentTransition = UISymbolContentTransition(.drawOff)
...
configuration.symbolContentTransition = UISymbolContentTransition(.drawOn)
...
configuration.symbolContentTransition = UISymbolContentTransition(.drawOn)
```
![[images/Screenshot 2025-06-11 at 11.33.38 PM.png]]![[images/Screenshot 2025-06-11 at 11.34.26 PM.png]]
