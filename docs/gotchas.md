# iPhone Duo gotchas

These are the ways apps break on iPhone Duo, each with a fix and the Apple source behind it. API names were checked against Apple's iOS 27.1 SDK reference pages in October 2026.

Items marked 🔎 are detected automatically by [FoldLint](https://github.com/divyaravitech/foldlint).

**Layout:** [1](#1-branching-on-idiom--idiom-branching) · [2](#2-uiscreenmain--uiscreen-main) · [3](#3-sizing-from-the-screen--screen-bounds-layout) · [4](#4-layout-driven-by-orientation--orientation-branching-orientation-lock) · [5](#5-assuming-symmetric-safe-areas) · [6](#6-content-under-the-fold-or-the-inner-camera)
**Bars:** [7](#7-standalone-bars--standalone-bar) · [8](#8-text-only-toolbar-items--text-only-bar-item) · [9](#9-toggling-vertical-bars-per-screen)
**Arrangements and hinge:** [10](#10-arrangementview-in-the-wrong-place--arrangement-in-scroll-navigation-inside-arrangement) · [11](#11-using-hinge-data-for-layout)
**Cameras:** [12](#12-mirroring-crashes--video-mirroring) · [13](#13-trusting-position--front-camera-assumption) · [14](#14-asking-default-for-the-individual-cameras--camera-discovery) · [15](#15-expecting-4k-or-depth-from-the-virtual-front-camera) · [16](#16-reusing-a-rotation-coordinator) · [17](#17-camera-capture-accessory-pitfalls)
**App lifecycle:** [18](#18-no-scene-lifecycle--scene-lifecycle) · [19](#19-app-wide-window-lookups--app-wide-window) · [20](#20-relying-on-uirequiresfullscreen--requires-full-screen) · [21](#21-losing-state-on-fold)

---

## 1. Branching on idiom 🔎 `idiom-branching`

iPhone Duo reports `.phone`, but its inner display has **regular** width and height size classes. A check like `if idiom == .pad { showSidebar() }` never shows the sidebar on the inner display.

```swift
// Before
if UIDevice.current.userInterfaceIdiom == .pad { showSidebar() }

// After (UIKit)
if traitCollection.horizontalSizeClass == .regular { showSidebar() }

// After (SwiftUI)
@Environment(\.horizontalSizeClass) private var horizontalSizeClass
```

Source: [Prepare your app for iPhone Duo](https://developer.apple.com/videos/play/tech-talks/111461/), *Use size classes*.

## 2. `UIScreen.main` 🔎 `uiscreen-main`

`UIScreen.main` is **deprecated in iOS 26**, and on a device with two displays there's no single "main" screen.

```swift
// Before
let scale = UIScreen.main.scale

// After
let scale = traitCollection.displayScale         // SwiftUI: @Environment(\.displayScale)
let screen = view.window?.windowScene?.screen    // only if you truly need a UIScreen
```

Source: [`UIScreen.main`](https://developer.apple.com/documentation/uikit/uiscreen/main).

## 3. Sizing from the screen 🔎 `screen-bounds-layout`

Your app can run in Split View, next to a vertical bar, or split by the fold. Its size is almost never the screen's size.

```swift
// Before
cell.frame.size.width = window.screen.bounds.width

// After
cell.frame.size.width = collectionView.bounds.inset(by: collectionView.safeAreaInsets).width
```

Source: [HIG: Designing for iPhone Duo](https://developer.apple.com/design/human-interface-guidelines/designing-for-iphone-duo), *Dynamic layouts*.

## 4. Layout driven by orientation 🔎 `orientation-branching` `orientation-lock`

The outer display behaves like any other iPhone. The inner display **doesn't honor your supported interface orientations**: your app is scaled there and in Split View. Landscape no longer means "wide", so choose layouts by size class.

```swift
// Before
if windowScene.interfaceOrientation.isLandscape { useTwoColumns() }

// After
if traitCollection.horizontalSizeClass == .regular { useTwoColumns() }
```

Source: [Prepare your app for iPhone Duo](https://developer.apple.com/videos/play/tech-talks/111461/).

## 5. Assuming symmetric safe areas

Vertical bars sit along one edge. In Split View, each app puts its bars on its outer edge, so the left app's bars are on the left. Never assume opposite insets are equal.

```swift
// Before
let width = view.bounds.width - view.safeAreaInsets.left * 2

// After
let width = view.bounds.inset(by: view.safeAreaInsets).width
```

Source: [Prepare your app for iPhone Duo](https://developer.apple.com/videos/play/tech-talks/111461/), *Respect safe areas*; [HIG](https://developer.apple.com/design/human-interface-guidelines/designing-for-iphone-duo), *Vertical controls*.

## 6. Content under the fold or the inner camera

When the device is partially open, the fold is a **division** region. While the inner camera is active, it's an **occlusion** region. Alerts, menus, sheets and split views adapt on their own. Your custom controls don't.

```swift
GeometryReader { proxy in
    let fold = proxy.reservedRegions(kind: .division).filter(\.isActive)
    // Keep buttons out of fold.map(\.frame). Scrolling content can pass under it.
}
```

Source: [ReservedRegion](https://developer.apple.com/documentation/swiftui/reservedregion); [Strike a pose with adaptive layouts on iPhone Duo](https://developer.apple.com/videos/play/tech-talks/111463/).

## 7. Standalone bars 🔎 `standalone-bar`

Only bars managed by containers (`UINavigationController`, `UITabBarController`, SwiftUI's `NavigationStack`, `TabView` and `.toolbar`) move to the vertical axis. A `UIToolbar()` you add yourself stays horizontal and can collide with the system's vertical bar.

```swift
// Before
let toolbar = UIToolbar()
view.addSubview(toolbar)

// After
navigationController?.isToolbarHidden = false
toolbarItems = [shareItem, .flexibleSpace(), deleteItem]
```

Source: [Raise the bar with iPhone Duo](https://developer.apple.com/videos/play/tech-talks/111462/).

## 8. Text-only toolbar items 🔎 `text-only-bar-item`

"Labels that include text stay in a horizontal bar, so prefer a symbol wherever one works." Give each item a title **and** a symbol: the system uses the symbol in the vertical bar and the title in overflow menus.

```swift
// Before
.toolbar { Button("Share") { share() } }

// After
.toolbar { Button("Share", systemImage: "square.and.arrow.up") { share() } }

// UIKit
UIBarButtonItem(title: "Share", image: UIImage(systemName: "square.and.arrow.up"), primaryAction: shareAction)
```

Source: [HIG: Designing for iPhone Duo](https://developer.apple.com/design/human-interface-guidelines/designing-for-iphone-duo), *Vertical controls*.

## 9. Toggling vertical bars per screen

`.toolbarVerticalBehavior(.disabled)` is for screens that are better with horizontal bars, such as a full-screen player or a calculator-style layout. Apple says to treat it as a stable choice and not toggle it as people navigate. To hide bars, use `toolbarVisibility(_:for:)`. In UIKit, `preferredVerticalBarBehavior` is a **read-only** property: you override it in your view controller rather than assigning it.

```swift
override var preferredVerticalBarBehavior: UIVerticalBarBehavior { .disabled }
```

Source: [toolbarVerticalBehavior(_:)](https://developer.apple.com/documentation/swiftui/view/toolbarverticalbehavior(_:)); [preferredVerticalBarBehavior](https://developer.apple.com/documentation/uikit/uiviewcontroller/preferredverticalbarbehavior).

## 10. `ArrangementView` in the wrong place 🔎 `arrangement-in-scroll` `navigation-inside-arrangement`

"An arrangement view lays out content but doesn't handle navigation, so place navigation containers like navigation split views and tab views around it rather than within it." Don't embed arrangements in lists or scroll views either.

```swift
// Before
List { ArrangementView { Map() } secondary: { Details() } }
ArrangementView { NavigationStack { Feed() } } secondary: { Player() }

// After
NavigationStack {
    ArrangementView { Map() } secondary: { Details() }
        .arrangementViewStyle(.split)
}
```

Source: [HIG](https://developer.apple.com/design/human-interface-guidelines/designing-for-iphone-duo), *Arrangement views*; [Strike a pose with adaptive layouts on iPhone Duo](https://developer.apple.com/videos/play/tech-talks/111463/), *When not to use an arrangement*.

## 11. Using hinge data for layout

`onHingeChange` and `UIHingeInteraction` are meant for **effects**, like the pitch-bend demo in Apple's talk. Use reserved regions and arrangements for layout. `hinge` is `nil` on devices without one, and in UIKit when the view leaves a hierarchy that provides hinge updates.

```swift
@State private var hinge: DeviceHinge?

var body: some View {
    Instrument(bend: hinge.map { $0.angle.degrees } ?? 0)
        .onHingeChange { _, new in hinge = new.hinge }   // status: .closed, .partiallyOpen, .fullyOpen
}
```

Source: [onHingeChange(isEnabled:_:)](https://developer.apple.com/documentation/swiftui/view/onhingechange(isenabled:_:)); [Leverage multiple displays and scenes on iPhone Duo](https://developer.apple.com/videos/play/tech-talks/111464/), *Hinge data versus layout APIs*.

## 12. Mirroring crashes 🔎 `video-mirroring`

"Set `automaticallyAdjustsVideoMirroring` to `false` before you assign `isVideoMirrored`, because assigning it while the connection adjusts mirroring on its own raises an exception." Assigning it when `isVideoMirroringSupported` is `false` also raises an exception. On Duo you override mirroring whenever a camera's position and direction disagree (gotcha 13), so this crash is easier to hit.

```swift
guard connection.isVideoMirroringSupported else { return }
connection.automaticallyAdjustsVideoMirroring = false
connection.isVideoMirrored = isFacingForward
```

Re-apply mirroring after you swap inputs: a new input creates a new preview connection that doesn't carry your override.

Source: [Choosing a camera by the direction it faces](https://developer.apple.com/documentation/avkit/choosing-a-camera-by-the-direction-it-faces).

## 13. Trusting `position` 🔎 `front-camera-assumption`

`position` describes where a camera **sits**, not which way it **faces**. On Duo, a front camera can face away from the user, and a rear camera can face them (that's how someone takes a selfie with the rear cameras). Use `AVCaptureDeviceDirectionCoordinator` from AVKit:

- Create it on the main actor with the view that shows your preview and **every** built-in camera you capture from, including rear cameras. Use the two physical front cameras, not the virtual one.
- Its change handler runs on the main actor. Don't call AVFoundation from it. Pass the `Sendable` `AVCaptureDeviceDescriptor` to the actor that owns your session, and resolve it there with `AVCaptureDevice(uniqueID:)`, which can return `nil`.
- Decide mirroring from the direction map, not from `position`.

Source: [Choosing a camera by the direction it faces](https://developer.apple.com/documentation/avkit/choosing-a-camera-by-the-direction-it-faces).

## 14. Asking `default` for the individual cameras 🔎 `camera-discovery`

`.builtInOuterUltraWideCamera` and `.builtInInnerUltraWideCamera` "may only be discovered using an AVCaptureDeviceDiscoverySession". `AVCaptureDevice.default(_:for:position:)` won't return them.

```swift
let session = AVCaptureDevice.DiscoverySession(
    deviceTypes: [.builtInOuterUltraWideCamera, .builtInInnerUltraWideCamera],
    mediaType: .video,
    position: .unspecified
)
```

Source: [builtInOuterUltraWideCamera](https://developer.apple.com/documentation/avfoundation/avcapturedevice/devicetype-swift.struct/builtinouterultrawidecamera).

## 15. Expecting 4K or depth from the virtual front camera

Existing front-camera code keeps working: on Duo, a discovery session for `.builtInWideAngleCamera` or `.builtInUltraWideCamera` with `position: .front` returns a **virtual front camera** that follows the active display. It only offers what both physical cameras share: **1080p at 60 fps, with no depth**. For the outer camera's 4K at 120 fps, or for depth, capture from the individual cameras and handle switching yourself.

Source: [Choosing a camera by the direction it faces](https://developer.apple.com/documentation/avkit/choosing-a-camera-by-the-direction-it-faces); [Build a great camera experience for iPhone Duo](https://developer.apple.com/videos/play/tech-talks/111465/).

## 16. Reusing a rotation coordinator

`AVCaptureDevice.RotationCoordinator` reports angles for the device you created it with, and on Duo those angles also change as your app moves between displays. **Create a new one each time you switch cameras.** After adopting it, set `photoOutput.isCameraSensorOrientationCompensationEnabled = false`.

Source: [Choosing a camera by the direction it faces](https://developer.apple.com/documentation/avkit/choosing-a-camera-by-the-direction-it-faces); [Build a great camera experience for iPhone Duo](https://developer.apple.com/videos/play/tech-talks/111465/).

## 17. Camera capture accessory pitfalls

The capture accessory, which shows content such as a script on the outer display while you record, comes with a few rules:

- **The system decides when to show it.** That requires the device to be open, your app in the foreground, and a capture session running. Treat the accessory as an enhancement, and keep every essential control in your main capture UI.
- **Scene manifest entries have no effect.** Accessory scenes have no project-level configuration. If one scene delegate handles several kinds of scene, check for the `windowCameraCaptureAccessory` session role.
- **Hold the registration strongly** (UIKit). To turn the content off, set `isEnabled = false`. Unregister only when your app stops offering it.
- **Test on a device.** Simulator has no camera.

Source: [Registering a camera capture accessory on iPhone Duo](https://developer.apple.com/documentation/avfoundation/registering-a-camera-capture-accessory-on-iphone-duo).

## 18. No scene lifecycle 🔎 `scene-lifecycle`

"In the next major release following iOS 26, UIScene lifecycle will be required when building with the latest SDK; otherwise, your app won't launch." iPhone Duo's multiple windows depend on scenes too. To adopt it, add a `UIApplicationSceneManifest` (or implement `application(_:configurationForConnecting:options:)`) and move window setup into a `UIWindowSceneDelegate`.

Source: [TN3187: Migrating to the UIKit scene-based life cycle](https://developer.apple.com/documentation/technotes/tn3187-migrating-to-the-uikit-scene-based-life-cycle).

## 19. App-wide window lookups 🔎 `app-wide-window`

With several scenes on the inner display, `UIApplication.shared.keyWindow`, `.windows` and `connectedScenes.first` can all return the wrong window. (`keyWindow` and `windows` are also deprecated.) Use `view.window`, or the scene you were given. You can create new windows only on the inner display, so handle failed scene requests. `UIWindowScene.ActivationAction` hides itself automatically when a new window isn't possible.

Source: [Leverage multiple displays and scenes on iPhone Duo](https://developer.apple.com/videos/play/tech-talks/111464/).

## 20. Relying on `UIRequiresFullScreen` 🔎 `requires-full-screen`

"iPhone Duo will continue to honor the UIRequiresFullScreen key, but your app will still resize when someone opens or closes their iPhone Duo."

Source: [Prepare your app for iPhone Duo](https://developer.apple.com/videos/play/tech-talks/111461/).

## 21. Losing state on fold

Opening or closing the device is a size change, not a relaunch. Keep functionality and state the same on both displays. Don't tear down view controllers, players or network sessions when the size changes.

Source: [HIG: Designing for iPhone Duo](https://developer.apple.com/design/human-interface-guidelines/designing-for-iphone-duo), *Best practices*.
