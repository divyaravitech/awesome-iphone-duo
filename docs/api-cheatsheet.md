# iPhone Duo API cheat sheet

Every entry below was checked against Apple's API reference pages for the iOS 27.1 SDK in October 2026. "27.1" means the API is new in the iOS 27.1 SDK, which was still in beta when this sheet was checked. Some APIs are older and simply matter more on Duo; their availability is marked.

## Layout around the fold

| Need | SwiftUI | UIKit | Since |
| --- | --- | --- | --- |
| Find the fold, the active inner camera or other reserved areas | `GeometryProxy.reservedRegions(kind:options:layoutDirectionBehavior:)` with kind `.division` (fold) or `.occlusion` (camera, Dynamic Island) | `UIView.reservedRegions(kind:options:)` | 27.1 |
| What a region tells you | `ReservedRegion`: `frame`, `isActive`, `kind`, `margins` | `UIView.ReservedRegion`: `frame`, `isActive`, `kind`, `margins` | 27.1 |
| Two views that rearrange around the fold | `ArrangementView { primary } secondary: { secondary }` | `UIArrangementViewController` + `setViewController(_:for:animated:)` | 27.1 |
| Choose the arrangement | `.arrangementViewStyle(.split)` or `.overlay`; restrict with `.split.axes(.horizontal)` | `updateArrangement(.split.axes(.horizontal), animated:)` (`UISplitArrangement`, `UIOverlayArrangement`) | 27.1 |
| Size the split | `.splitArrangementLayoutRatio(_:)`, `.splitArrangementLayoutSize(…)`, `.splitArrangementFixedLayoutSize(horizontal:vertical:)` | — | 27.1 |
| List/detail that adapts automatically | `NavigationSplitView` | `UISplitViewController` | existing |
| Sidebar on the inner display | `TabView { … }.tabViewStyle(.sidebarAdaptable).defaultTabBarPlacement(.sidebar)` | `tabBarController.sidebar.preferredPlacement = .sidebar` | 27.0 |
| Size decisions | `@Environment(\.horizontalSizeClass)` | `traitCollection.horizontalSizeClass` | existing |
| Display scale without `UIScreen.main` | `@Environment(\.displayScale)` | `traitCollection.displayScale` | existing |

> Right-to-left layouts mirror region frames unless you pass `layoutDirectionBehavior: .fixed`.

## Hinge (for effects, not layout)

| Need | SwiftUI | UIKit | Since |
| --- | --- | --- | --- |
| Observe the hinge | `.onHingeChange { old, new in … }` (`DeviceHingeContext`) | `UIHingeInteraction { interaction, update in … }` added with `view.addInteraction(_:)` | 27.1 |
| Current hinge | `new.hinge` → `DeviceHinge?` (`nil` without a hinge) | `update.hinge` → `UIHinge?` | 27.1 |
| Hinge state | `hinge.status`: `.closed`, `.partiallyOpen`, `.fullyOpen`; `hinge.angle`: `Angle` | `hinge.status`, `hinge.angle` | 27.1 |

## Vertical bars

| Need | SwiftUI | UIKit | Since |
| --- | --- | --- | --- |
| Which edge the vertical bar uses | `@Environment(\.toolbarVerticalEdge)` → `HorizontalEdge?` | `traitCollection.verticalBarEdge` → `UIVerticalBarEdge` | 27.1 |
| Opt a screen out of vertical bars | `.toolbarVerticalBehavior(.disabled)` | override `preferredVerticalBarBehavior` (read-only) to return `.disabled` | 27.1 |
| Keep an item horizontal or prefer vertical | `ToolbarItem { … }.axisBehavior(.horizontalOnly / .verticalPreferred)` | `UIBarButtonItem.axisBehavior` | 27.1 |
| Which bar compresses first | `.toolbarVerticalCompressionBehavior(.prefersTabBar / .prefersToolbarItems)` | `navigationItem.verticalBarCompressionBehavior = .prefersTabBar / .prefersBarItems` | 27.1 |
| What overflows first | `ToolbarItem { … }.visibilityPriority(.high)` | `UIBarButtonItem.visibilityPriority` | 27.0 |
| System overflow menu | `ToolbarOverflowMenu { … }` | `navigationItem.additionalOverflowItems` (`UIDeferredMenuElement`) | 27.0 / 16.0 |

## Cameras

| Need | API | Since |
| --- | --- | --- |
| Front camera that follows the active display | `AVCaptureDevice.DiscoverySession(deviceTypes: [.builtInWideAngleCamera, .builtInUltraWideCamera], mediaType: .video, position: .front)` returns the **virtual front camera** on Duo (`isVirtualDevice == true`; `activePrimaryConstituent` is the streaming camera) | existing |
| Each front camera individually | `.builtInOuterUltraWideCamera`, `.builtInInnerUltraWideCamera`, discoverable **only** through `AVCaptureDevice.DiscoverySession` | 27.1 |
| Which cameras face the user right now | `AVCaptureDeviceDirectionCoordinator(view:deviceTypes:changeHandler:)` (AVKit) → `AVCaptureDeviceDirectionMap.forwardFacingDeviceDescriptors` / `backwardFacingDeviceDescriptors` | 27.1 |
| Pass a camera between actors | `AVCaptureDeviceDescriptor` (`Sendable`): resolve with `AVCaptureDevice(uniqueID:)` on your capture actor | 27.1 |
| Upright preview and output across displays | `AVCaptureDevice.RotationCoordinator`, one per device; create a new one when you switch cameras | 17.0 |
| Turn off sensor-orientation compensation after adopting the rotation coordinator | `AVCapturePhotoOutput.isCameraSensorOrientationCompensationEnabled = false` | existing |
| Override mirroring safely | check `isVideoMirroringSupported`, set `automaticallyAdjustsVideoMirroring = false`, then `isVideoMirrored` | existing |

### Camera capabilities

| Camera | How to get it | Max resolution | Max frame rate | Depth |
| --- | --- | --- | --- | --- |
| Virtual front | Discovery session, `position: .front` | 1080p | 60 fps | No |
| Inner front (under the display) | `.builtInInnerUltraWideCamera` | 1080p | 60 fps | Yes |
| Outer front | `.builtInOuterUltraWideCamera` | 4K | 120 fps | Yes |

Capabilities from Apple's [Build a great camera experience for iPhone Duo](https://developer.apple.com/videos/play/tech-talks/111465/).

## Second-display content during capture

| Need | SwiftUI | UIKit | Since |
| --- | --- | --- | --- |
| Show content (script, countdown, framing) on the outer display while capturing | `.sceneAccessory { CameraCaptureAccessory { … } }` | `registerSceneAccessory(UISceneAccessory.cameraCapture(sceneConfiguration:userInfo:))` on the capture view controller → `UISceneAccessoryRegistration` | 27.1 |
| Know whether the system can show it | `.onAvailabilityChange { isAvailable in … }` | `registration.isAvailable` (observable) | 27.1 |
| Let people turn it off | `CameraCaptureAccessory(isEnabled: $binding) { … }` | `registration.isEnabled = false` | 27.1 |

## Sources

[ReservedRegion](https://developer.apple.com/documentation/swiftui/reservedregion) ·
[UIView.ReservedRegion](https://developer.apple.com/documentation/uikit/uiview/reservedregion) ·
[ArrangementView](https://developer.apple.com/documentation/swiftui/arrangementview) ·
[UIArrangementViewController](https://developer.apple.com/documentation/uikit/uiarrangementviewcontroller) ·
[onHingeChange](https://developer.apple.com/documentation/swiftui/view/onhingechange(isenabled:_:)) ·
[DeviceHinge](https://developer.apple.com/documentation/swiftui/devicehinge) ·
[UIHingeInteraction](https://developer.apple.com/documentation/uikit/uihingeinteraction) ·
[toolbarVerticalBehavior](https://developer.apple.com/documentation/swiftui/view/toolbarverticalbehavior(_:)) ·
[toolbarVerticalEdge](https://developer.apple.com/documentation/swiftui/environmentvalues/toolbarverticaledge) ·
[ToolbarVerticalCompressionBehavior](https://developer.apple.com/documentation/swiftui/toolbarverticalcompressionbehavior) ·
[UIVerticalBarCompressionBehavior](https://developer.apple.com/documentation/uikit/uiverticalbarcompressionbehavior) ·
[preferredVerticalBarBehavior](https://developer.apple.com/documentation/uikit/uiviewcontroller/preferredverticalbarbehavior) ·
[verticalBarEdge](https://developer.apple.com/documentation/uikit/uitraitcollection/verticalbaredge) ·
[axisBehavior](https://developer.apple.com/documentation/swiftui/toolbarcontent/axisbehavior(_:)) ·
[visibilityPriority](https://developer.apple.com/documentation/swiftui/toolbarcontent/visibilitypriority(_:)) ·
[ToolbarOverflowMenu](https://developer.apple.com/documentation/swiftui/toolbaroverflowmenu) ·
[defaultTabBarPlacement](https://developer.apple.com/documentation/swiftui/view/defaulttabbarplacement(_:)) ·
[Choosing a camera by the direction it faces](https://developer.apple.com/documentation/avkit/choosing-a-camera-by-the-direction-it-faces) ·
[Registering a camera capture accessory on iPhone Duo](https://developer.apple.com/documentation/avfoundation/registering-a-camera-capture-accessory-on-iphone-duo) ·
[CameraCaptureAccessory](https://developer.apple.com/documentation/swiftui/cameracaptureaccessory)
