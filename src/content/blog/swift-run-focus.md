---
title: "Fixing keyboard input in terminal SwiftUI apps"
description: "A tale of terminal launches, activation policies, and the mild existential crisis of a text field that refuses to accept input."
date: 2025-12-10
tags: ["swift", "swiftui", "macos", "debugging"]
---

*A tale of terminal launches, activation policies, and the mild existential crisis of a text field that refuses to accept input.*

---

## The Problem

Picture this: you've spent hours crafting a beautiful SwiftUI app. The blur effects are *chef's kiss*. The animations butter-smooth. You run `swift build && .build/debug/MyApp` from your terminal, the window appears in all its glory, you click on your lovingly styled TextField, you start typing and...

Nothing. The characters appear in your terminal instead.

You click the TextField again. Still nothing. You question your career choices. You wonder if the computer has finally achieved sentience and is simply refusing to cooperate.

I've been there. Let me save you the hour of confused Googling.

## What's Actually Happening

When you build a macOS app with Swift Package Manager and run it directly from the terminal, macOS doesn't recognize it as a "real" GUI application. Without the familiar `.app` bundle structure (the one with `Info.plist`, `Resources`, and all that ceremony), macOS treats your executable as a command-line tool that happens to have windows.

And command-line tools? They don't get keyboard focus. The terminal that spawned them keeps it.

Your TextField isn't broken. It's just being ghosted by the operating system.

## The Fix

Two lines of code. That's it. I spent an embarrassingly long time on this so you don't have to.

### Step 1: Tell macOS This Is a Real App

In your `App` struct's `init()`, add:

```swift
#if os(macOS)
import AppKit
#endif

@main
struct MyApp: App {
    init() {
        #if os(macOS)
        // Tell macOS this is a proper GUI app, not a terminal pretender
        NSApplication.shared.setActivationPolicy(.regular)
        #endif

        // ... rest of your init
    }
}
```

The `.regular` activation policy tells macOS: "Yes, I know I emerged from a terminal like some kind of command-line creature, but I have windows and dreams and I deserve to be in the Dock."

### Step 2: Seize Focus Like You Mean It

In your root view's `.onAppear`, activate the app:

```swift
var body: some Scene {
    WindowGroup {
        ContentView()
            #if os(macOS)
            .onAppear {
                // Grab focus from the terminal that spawned us
                NSApplication.shared.activate(ignoringOtherApps: true)
            }
            #endif
    }
}
```

The `ignoringOtherApps: true` parameter is delightfully aggressive. It's the code equivalent of walking into a room and announcing "I'm the main character now."

## Complete solution

Here's a full example for the copy-paste inclined:

```swift
import SwiftUI
#if os(macOS)
import AppKit
#endif

@main
struct MyApp: App {
    init() {
        #if os(macOS)
        // Force app to become active GUI app when launched from terminal
        // Without this, keyboard input goes to terminal instead of the app
        NSApplication.shared.setActivationPolicy(.regular)
        #endif
    }

    var body: some Scene {
        WindowGroup {
            ContentView()
                #if os(macOS)
                .onAppear {
                    // Activate app and bring to front when launched from terminal
                    NSApplication.shared.activate(ignoringOtherApps: true)
                }
                #endif
        }
    }
}
```

## Why Xcode works differently

If you build and run from Xcode (⌘R), you'll never encounter this issue. Xcode creates a proper `.app` bundle with all the metadata macOS expects. The `Info.plist` declares your app's identity, the bundle structure signals legitimacy, and macOS treats it with the respect a GUI application deserves.

Swift Package Manager, bless its heart, just creates a naked executable. Efficient, portable, but socially awkward at the macOS party.

## When you'll encounter this

- Running `swift build && .build/debug/YourApp` during development
- Building with SPM for quick iteration without Xcode
- Any scenario where you're launching a GUI app from a terminal process

## A pragmatic solution

Is this the most elegant solution? No. The elegant solution is to build a proper `.app` bundle with an `Info.plist` that declares your `LSUIElement` status and activation policy. But sometimes you're iterating quickly, or you're building a tool that needs to work both ways, or you just want to ship the thing.

The code above works. It's two lines. Sometimes pragmatism beats elegance.

---

*Now go forth and type into your text fields with confidence. They're listening.*
