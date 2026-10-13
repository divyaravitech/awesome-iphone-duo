# iPhone Duo migration playbook

Copy this into a GitHub issue and tick items off as you go. The links point to the [gotchas](gotchas.md) and the [API cheat sheet](api-cheatsheet.md).

> **Key dates:** iPhone Duo is available from **October 23, 2026** and ships with iOS 27.1. Apple opened App Store submissions for Duo-optimized apps built with **Xcode 27.1** in early October 2026. Press reports say Apple will require iPhone Duo screenshots for all app submissions from **April 2027**; confirm the date in App Store Connect.

## Phase 0: Audit (about an hour)

- [ ] Run [FoldLint](https://github.com/divyaravitech/foldlint) and record the readiness score as your baseline (`foldlint --write-baseline .foldlint-baseline.json`)
- [ ] List screens with fixed widths, hard-coded device sizes or custom bars
- [ ] Confirm the app uses the UIScene lifecycle ([gotcha 18](gotchas.md#18-no-scene-lifecycle--scene-lifecycle))
- [ ] List camera features: preview, capture, mirroring, depth, front/back switching

## Phase 1: Build with the new SDK

- [ ] Install Xcode 27.1 and build with the iOS 27.1 SDK. Only builds against 27.1 get edge-to-edge layout on the inner display, vertical bars and the fold APIs.
- [ ] Run in **Device Hub**: open, close, rotate and partially fold the device
- [ ] Try Xcode 27.1's **App Resizability** skill on your UIKit and SwiftUI code

## Phase 2: Layout

- [ ] Replace idiom checks with size classes ([gotcha 1](gotchas.md#1-branching-on-idiom--idiom-branching))
- [ ] Remove `UIScreen.main` ([gotcha 2](gotchas.md#2-uiscreenmain--uiscreen-main))
- [ ] Size from the view, scene or `GeometryReader`, never the screen ([gotcha 3](gotchas.md#3-sizing-from-the-screen--screen-bounds-layout))
- [ ] Handle each safe-area inset separately, and test with bars on **both** edges ([gotcha 5](gotchas.md#5-assuming-symmetric-safe-areas))
- [ ] Use `NavigationSplitView` / `UISplitViewController` for list and detail
- [ ] On the inner display, consider a sidebar: `.tabViewStyle(.sidebarAdaptable).defaultTabBarPlacement(.sidebar)`
- [ ] Keep custom controls out of reserved regions: the fold (**division**) and the active inner camera (**occlusion**) ([gotcha 6](gotchas.md#6-content-under-the-fold-or-the-inner-camera))
- [ ] Prefer an even number of grid columns, so content divides cleanly at the fold
- [ ] Use `ArrangementView` where a layout already looks like an `HStack`, `VStack` or `ZStack` of two parts, with navigation outside it ([gotcha 10](gotchas.md#10-arrangementview-in-the-wrong-place--arrangement-in-scroll-navigation-inside-arrangement))

## Phase 3: Bars

- [ ] Use container-managed bars, not standalone `UIToolbar`, `UINavigationBar` or `UITabBar` ([gotcha 7](gotchas.md#7-standalone-bars--standalone-bar))
- [ ] Give every toolbar item a title and a symbol ([gotcha 8](gotchas.md#8-text-only-toolbar-items--text-only-bar-item))
- [ ] Order items: Back or Close first, then prominent actions such as Done
- [ ] Set `visibilityPriority` for frequently used items and items with badges
- [ ] Move custom overflow menus into `ToolbarOverflowMenu` / `additionalOverflowItems`
- [ ] Choose a compression behavior: navigation-focused apps keep the tab bar, task-focused apps keep toolbar items
- [ ] Opt out of vertical bars only for screens that need it, and as a stable choice ([gotcha 9](gotchas.md#9-toggling-vertical-bars-per-screen))

## Phase 4: Multitasking and scenes

- [ ] Test Split View at every width, and with pinned Picture in Picture
- [ ] Support multiple windows on the inner display, and handle failed scene requests ([gotcha 19](gotchas.md#19-app-wide-window-lookups--app-wide-window))
- [ ] Preserve state across fold and unfold: no reloads and no lost input ([gotcha 21](gotchas.md#21-losing-state-on-fold))

## Phase 5: Camera (if applicable)

- [ ] Keep discovering the front camera as before. On Duo you get the virtual front camera, limited to 1080p/60 with no depth ([gotcha 15](gotchas.md#15-expecting-4k-or-depth-from-the-virtual-front-camera))
- [ ] If capture is your app's main job, use the individual cameras through a discovery session ([gotcha 14](gotchas.md#14-asking-default-for-the-individual-cameras--camera-discovery))
- [ ] Adopt `AVCaptureDeviceDirectionCoordinator`, and reconfigure on your capture actor ([gotcha 13](gotchas.md#13-trusting-position--front-camera-assumption))
- [ ] Mirror from the direction map, with the safe mirroring sequence ([gotcha 12](gotchas.md#12-mirroring-crashes--video-mirroring))
- [ ] Create a new `RotationCoordinator` per camera, then disable sensor-orientation compensation ([gotcha 16](gotchas.md#16-reusing-a-rotation-coordinator))
- [ ] Consider a `CameraCaptureAccessory` for a script, countdown or framing preview on the outer display ([gotcha 17](gotchas.md#17-camera-capture-accessory-pitfalls))
- [ ] Test on a real device, because Simulator has no camera

## Phase 6: Ship

- [ ] Capture App Store screenshots for both displays ([screenshot specifications](https://developer.apple.com/help/app-store-connect/reference/app-information/screenshot-specifications))
- [ ] Add FoldLint to CI with `--baseline` (fail on new issues) and, optionally, `--min-score`
- [ ] Add the "iPhone Duo ready" badge to your README

---

## Extra steps by app type

**Feeds and social.** Scrolling content can pass under the fold, but floating buttons and composers can't. On the inner display, show detail beside the feed, the way Mail shows a list and a message side by side.

**Media and video.** The inner display is close to square (about 1.42:1, by [one developer's estimate](https://blakecrosley.com/blog/iphone-duo-for-developers) from the screenshot sizes), so 16:9 video letterboxes there. Consider a layout where controls or related content use the extra space. Check Picture in Picture placement.

**Documents and readers.** Lay out a two-page spread around the fold. Keep the selection and the cursor when the device folds.

**Camera and creator apps.** Show a teleprompter or framing preview on the outer display. Mirror correctly when a rear camera faces the user.

**Games.** You can lock to portrait or landscape, but fill the screen in every pose. Prefer changing the aspect ratio over letterboxing; if you must letterbox, add artwork to the padding.
