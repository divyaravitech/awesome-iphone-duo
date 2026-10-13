# Awesome iPhone Duo [![Awesome](https://awesome.re/badge.svg)](https://awesome.re)

> A curated list of resources, APIs, gotchas and tools for building apps for iPhone Duo, Apple's first foldable iPhone.

iPhone Duo has a 5.4-inch outer display and a 7.6-inch inner display that folds. It's available from October 23, 2026 and runs iOS 27.1. Unmodified iPhone apps still run on it. Rebuilding with Xcode 27.1 and the iOS 27.1 SDK is what gets an app edge-to-edge layout on the inner display, vertical bars and the fold APIs.

API names in this repo were checked against Apple's reference documentation in October 2026, and CI checks every link weekly. iOS 27.1 is new, so if something has changed, please [open a correction](https://github.com/divyaravitech/awesome-iphone-duo/issues/new?template=correction.yml).

## Contents

- [Guides](#guides)
- [Official Apple Resources](#official-apple-resources)
  - [Documentation](#documentation)
  - [Tech Talks](#tech-talks)
  - [Labs and Q&A](#labs-and-qa)
  - [App Store](#app-store)
- [Tools and Libraries](#tools-and-libraries)
  - [Native](#native)
  - [Flutter](#flutter)
  - [Simulators](#simulators)
- [Articles](#articles)
- [Design](#design)
- [Web](#web)
- [Prior Art From Other Foldables](#prior-art-from-other-foldables)
- [News](#news)

## Guides

These guides live in this repo. Each claim in them links to its Apple source.

- [Migration Playbook](https://github.com/divyaravitech/awesome-iphone-duo/blob/main/docs/playbook.md) - A checklist you can paste into an issue, organised by phase (audit, build, layout, bars, scenes, camera, ship), with extra steps by app type.
- [Gotchas](https://github.com/divyaravitech/awesome-iphone-duo/blob/main/docs/gotchas.md) - 21 ways apps break on Duo, from idiom checks and `UIScreen.main` to mirroring exceptions and camera direction, each with a fix.
- [API Cheat Sheet](https://github.com/divyaravitech/awesome-iphone-duo/blob/main/docs/api-cheatsheet.md) - SwiftUI and UIKit APIs side by side, with availability, for reserved regions, arrangements, the hinge, vertical bars and cameras.

## Official Apple Resources

- [Get ready for iPhone Duo](https://developer.apple.com/iphone-duo/) - Apple's hub for videos, labs, tools and documentation.
- [Prepare](https://developer.apple.com/iphone-duo/prepare/) - Apple's preparation page.

### Documentation

- [Preparing your app for iPhone Duo](https://developer.apple.com/documentation/technologyoverviews/preparing-your-app-for-iphone-duo) - Technology overview and starting point.
- [Human Interface Guidelines: Designing for iPhone Duo](https://developer.apple.com/design/human-interface-guidelines/designing-for-iphone-duo) - Poses, reserved regions, arrangement views and vertical controls.
- [Choosing a camera by the direction it faces](https://developer.apple.com/documentation/avkit/choosing-a-camera-by-the-direction-it-faces) - The virtual front camera, `AVCaptureDeviceDirectionCoordinator`, and mirroring done safely.
- [Registering a camera capture accessory on iPhone Duo](https://developer.apple.com/documentation/avfoundation/registering-a-camera-capture-accessory-on-iphone-duo) - Show a script or framing preview on the outer display while recording.
- [ArrangementView](https://developer.apple.com/documentation/swiftui/arrangementview) - The SwiftUI container that arranges two views around the fold.
- [ReservedRegion](https://developer.apple.com/documentation/swiftui/reservedregion) - Find the fold and the active inner camera in a view's coordinate space.
- [TN3187: Migrating to the UIKit scene-based life cycle](https://developer.apple.com/documentation/technotes/tn3187-migrating-to-the-uikit-scene-based-life-cycle) - The UIScene lifecycle becomes mandatory with the SDK after iOS 26.
- [Apple Design Resources](https://developer.apple.com/design/resources/) - Templates with margins and safe areas.

### Tech Talks

- [Design for iPhone Duo](https://developer.apple.com/videos/play/tech-talks/111466/) - Design principles, control positioning, the outer and inner displays, sheets and fold avoidance.
- [Prepare your app for iPhone Duo](https://developer.apple.com/videos/play/tech-talks/111461/) - Building with the 27.1 SDK, size classes, avoiding screen assumptions, safe areas and reserved regions.
- [Raise the bar with iPhone Duo](https://developer.apple.com/videos/play/tech-talks/111462/) - Vertical toolbars and tab bars, item axis, the overflow menu and visibility priority.
- [Strike a pose with adaptive layouts on iPhone Duo](https://developer.apple.com/videos/play/tech-talks/111463/) - Division and occlusion regions, and split and overlay arrangements.
- [Leverage multiple displays and scenes on iPhone Duo](https://developer.apple.com/videos/play/tech-talks/111464/) - Hinge angle, Split View, multiple scenes and scene accessories.
- [Build a great camera experience for iPhone Duo](https://developer.apple.com/videos/play/tech-talks/111465/) - The new front cameras, the direction coordinator, preview and rotation.

### Labs and Q&A

- [iPhone Duo Group Lab, Day 1](https://developer.apple.com/videos/play/meet-with-apple/285/) - Recorded lab with Apple engineers.
- [iPhone Duo Group Lab, Day 2](https://developer.apple.com/videos/play/meet-with-apple/286/) - Recorded lab with Apple engineers.
- [Popular Q&A highlights](https://apple.co/iPhoneDuo-QA01) - Apple's roundup of the most-asked questions.
- [Photos and camera Q&A](https://developer.apple.com/forums/activities/1659080) - Forum Q&A session.
- [SwiftUI Q&A](https://developer.apple.com/forums/activities/1664080) - Forum Q&A session.
- [UIKit Q&A](https://developer.apple.com/forums/activities/1670080) - Forum Q&A session.

### App Store

- [Screenshot specifications](https://developer.apple.com/help/app-store-connect/reference/app-information/screenshot-specifications) - Required screenshot sizes for each device.
- [App preview specifications](https://developer.apple.com/help/app-store-connect/reference/app-information/app-preview-specifications) - App preview video specs.
- [App Store asset best practices](https://developer.apple.com/app-store/asset-best-practices/) - Guidance on marketing assets.

## Tools and Libraries

### Native

- [FoldLint](https://github.com/divyaravitech/foldlint) - A static analyzer that scores a Swift project's iPhone Duo readiness, with rules that cite Apple's documentation, baselines for CI, SARIF output, a SwiftPM plugin and a GitHub Action.

### Flutter

- [nitro_fold_duo](https://pub.dev/packages/nitro_fold_duo) - Exposes fold geometry, hinge posture and the side control bar on iPhone Duo.
- [foldable](https://pub.dev/packages/foldable) - Exposes the iPhone Duo hinge angle, posture, and fold and camera regions to Dart, and does nothing on other devices.
- [iphone_duo_ui_pro](https://pub.dev/packages/iphone_duo_ui_pro) - Fold-aware two-pane layouts, hinge-aligned grids and navigation bars.

### Simulators

- [Duo](https://www.producthunt.com/p/duo-9/duo-536) - Unofficial desktop and web fold simulator with its own SDK. It doesn't run iOS apps; for that, use Device Hub in Xcode 27.1, which can open, close, rotate and partially fold the device.

## Articles

- [Seeing Double with the iPhone Duo](https://www.kodeco.com/54224360-seeing-double-with-the-iphone-duo) - Kodeco's overview of best practices, with code previews of the new APIs.
- [iPhone Duo: First Developer Good to Knows](https://swiftjectivec.com/iPhone-Duo-First-Developer-Good-to-Knows/) - API digest of Apple's tech talks.
- [iPhone Duo SDK Explained](https://ecorpit.com/iphone-duo-sdk-new-apis-xcode-27-1-developer-guide-2026/) - Inventory of the iOS 27.1 SwiftUI, UIKit and camera APIs, with their limitations.
- [Adapting Your iOS App to iPhone Duo: Two Layout Branches, Not a Rewrite](https://adapty.io/blog/markdown/adapt-your-ios-app-to-iphone-duo.md) - Why size classes cover most of the work.
- [iPhone Duo for Developers: The 1.42 Problem and the SDK Gap](https://blakecrosley.com/blog/iphone-duo-for-developers) - Screen geometry inferred from App Store Connect, and what it means for video and two-pane layouts.
- [How to make your paywall iPhone Duo-ready](https://www.revenuecat.com/blog/engineering/paywalls-on-iphone-duo) - RevenueCat on paywall layout per posture and close-button placement.
- [iPhone Duo has launched: What now for your app?](https://www.apadmi.com/insights/iphone-duo-what-now-for-your-app/) - An agency's guide to choosing how far to optimise.
- [The race to build great iPhone Duo apps is about to begin](https://9to5mac.com/2026/09/10/the-race-to-build-great-iphone-duo-apps-is-about-to-begin/) - 9to5Mac on why custom toolbars and navigation will lag at launch.

## Design

- [Designing for iPhone Duo: What Moves, Splits, and Stays](https://blakecrosley.com/blog/designing-for-iphone-duo) - A designer's walkthrough of the HIG.

## Web

- [Apple's $2K phone hides a CSS secret](https://www.stork.ai/blog/apples-2k-phone-hides-a-css-secret) - The `device-posture` media query and the Viewport Segments API on iPhone Duo.

## Prior Art From Other Foldables

- [Make your app fold aware](https://developer.android.com/develop/ui/compose/layouts/adaptive/foldables/make-your-app-fold-aware) - Android's tabletop and book postures in Jetpack Compose.
- [Microsoft Surface Duo docs](https://learn.microsoft.com/en-us/dual-screen/) - Dual-screen design patterns. The docs are archived but still useful.

## News

- [Apple unveils iPhone Duo](https://www.apple.com/bg/newsroom/2026/09/apple-unveils-iphone-duo/) - Apple Newsroom announcement with specs and dates.
- [Apple Publishes Developer Video Series for iPhone Duo](https://www.macrumors.com/2026/09/09/developer-video-series-for-iphone-duo/) - MacRumors on the six-video series.
- [iPhone Duo makes developers think in folds](https://devclass.com/devops/2026/09/16/apple-iphone-duo-makes-developers-think-in-folds/5296425) - DevClass on the developer impact.
- [Apple gives the okay to submit iPhone Duo apps](https://www.mactrast.com/2026/10/apple-give-okay-to-developers-to-submit-their-iphone-duo-apps/amp/) - Submissions open with Xcode 27.1, and iPhone Duo screenshots are required from April 2027.

## Contributing

Contributions are welcome. Read the [contribution guidelines](CONTRIBUTING.md) first.

*Apple, iPhone, Xcode, SwiftUI and UIKit are trademarks of Apple Inc. This project is not affiliated with Apple.*
