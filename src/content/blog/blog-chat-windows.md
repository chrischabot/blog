---
title: "Implementing an auto-scroll window view in SwiftUI"
description: "A journey through scroll behavior, IRC formatting codes from the 1990s, and the existential dread of LazyVStack."
date: 2025-12-10
tags: ["swift", "swiftui", "irc", "engineering"]
---

Building a chat window seems deceptively simple. It's just a list of messages, right? You scroll down, new messages appear at the bottom, Bob's your uncle. If only software development worked like that.

This is the story of how I built the chat interface for cIRC, my Swift/SwiftUI IRC client. It's a tale of hubris, humility, and learning to respect problems that looked trivial from a distance.

## The Humble Requirements

The chat window needed to do three things:

1. Show messages
2. Scroll to the bottom when new messages arrive (but only if you're already at the bottom)
3. Parse IRC formatting codes that predate most interns

Simple. I'd be done by lunch.

Reader, I was not done by lunch.

## Act I: The Scroll Position Debacle

SwiftUI's `ScrollView` is a marvel of declarative UI design. It's also, shall we say, *opinionated* about how much control it gives you over scroll position.

My first attempt was charmingly naive:

```swift
ScrollView {
    LazyVStack {
        ForEach(messages) { message in
            MessageRow(message: message)
        }
    }
}
```

This worked! Messages appeared. I could scroll. Victory was declared, champagne was uncorked, and then I asked myself: "Why doesn't it scroll down when new messages arrive?"

Ah.

### The "Am I At The Bottom?" Problem

The fundamental challenge of chat interfaces is this: when a new message arrives, you should scroll to show it—but *only* if the user was already at the bottom. If they've scrolled up to read history, you shouldn't rudely yank them back down like an overeager Labrador.

SwiftUI, in its infinite wisdom, doesn't expose scroll position directly. You can't just ask "hey, are we at the bottom?" You have to *infer* it, like a detective piecing together clues from a crime scene where the only witness is a `GeometryReader` that lies.

My solution involved the new `onScrollGeometryChange` modifier:

```swift
.onScrollGeometryChange(for: Bool.self) { geometry in
    let contentHeight = geometry.contentSize.height
    let containerHeight = geometry.containerSize.height
    let offsetY = geometry.contentOffset.y
    let distanceFromBottom = contentHeight - containerHeight - offsetY
    return distanceFromBottom <= bottomThreshold
} action: { _, isAtBottom in
    guard !isScrolling else { return }
    if isAtBottom != isPinnedToBottom {
        isPinnedToBottom = isAtBottom
    }
}
```

Elegant? Not particularly. Does it work? After approximately seventeen iterations, yes.

### The Recursive Scroll Problem

Here's where things got properly weird. I needed to scroll to the bottom when content grew, so I added:

```swift
.onScrollGeometryChange(for: CGFloat.self) { geometry in
    geometry.contentSize.height
} action: { oldHeight, newHeight in
    guard isPinnedToBottom else { return }
    guard newHeight > oldHeight else { return }
    proxy.scrollTo(bottomID, anchor: .bottom)
}
```

But scrolling changes the geometry. Which triggers the handler. Which checks if we should scroll. Which changes the geometry. You see where this is going.

The solution was a `isScrolling` flag that we set before programmatic scrolls and clear after a brief delay:

```swift
isScrolling = true
proxy.scrollTo(bottomID, anchor: .bottom)
DispatchQueue.main.asyncAfter(deadline: .now() + 0.05) {
    isScrolling = false
}
```

Is `0.05` seconds the correct delay? Empirically, yes. Theoretically, I have no idea. It works on my machine, and isn't that what software engineering is all about?

## Act II: The Invisible Bottom Anchor

Messages need to scroll *past* all content to the true bottom, including any padding. My first attempts kept leaving a few pixels of the last message cut off, like a bad haircut.

The fix was almost embarrassingly simple once I found it: an invisible anchor element:

```swift
LazyVStack {
    ForEach(buffer.messages) { message in
        MessageRow(message: message)
    }
}
.padding()

// The hero we needed
Color.clear
    .frame(height: 1)
    .id(bottomID)
```

One pixel of invisible color, carrying the entire scroll experience on its metaphorical shoulders.

## Act III: IRC Formatting, or How I Learned to Stop Worrying and Love Control Characters

IRC was born in 1988, which means its text formatting predates not just Unicode, but widespread agreement that control characters should be limited to actually controlling things.

Here's what I'm dealing with:

- `\u{02}` (Ctrl+B) - Bold
- `\u{03}` - Color codes (followed by numbers, naturally)
- `\u{1D}` - Italic
- `\u{1F}` - Underline
- `\u{0F}` - Reset everything
- `\u{16}` - Reverse video (swap foreground/background, because why not)

Color codes deserve special mention. The format is `\u{03}FG,BG` where FG and BG are 1-2 digit numbers mapping to a 16-color palette that was clearly designed by someone who thought "salmon pink" and "brown" were equally important choices.

```swift
private let ircColors: [Color] = [
    .white,                                      // 0
    .black,                                      // 1
    Color(red: 0, green: 0, blue: 0.5),          // 2 - navy
    Color(red: 0, green: 0.5, blue: 0),          // 3 - green
    .red,                                        // 4
    Color(red: 0.5, green: 0, blue: 0),          // 5 - brown/maroon
    .purple,                                     // 6
    .orange,                                     // 7
    // ... and so on, each more questionable than the last
]
```

### The Parser That Could

Parsing this required tracking state through the message:

```swift
var foreground: Color?
var background: Color?
var isBold = false
var isItalic = false
var isUnderline = false
var isStrikethrough = false

// Every character is a new adventure
switch char {
case "\u{03}":
    flushSegment()
    // Parse 1-2 digit foreground
    // Optionally parse comma + 1-2 digit background
    // Question your life choices
```

The real fun is that `\u{03}` with *no* following number resets colors, but `\u{03}4` sets foreground to red, and `\u{03}4,2` sets foreground to red and background to navy.

I briefly considered just stripping all formatting codes. Then I joined an IRC channel where someone's username was rendered in rainbow colors, and I knew I had to do this properly.

## Act IV: The Buffer and the Deque

Messages need to be stored efficiently. IRC channels can be chatty—thousands of messages aren't unusual. I capped the buffers at 2,000 messages using swift-collections' `Deque`:

```swift
public var messages: Deque<MessageState>

public func addMessage(_ message: MessageState, incrementUnread: Bool = true) {
    messages.append(message)

    if messages.count > Self.maxMessages {
        let excess = messages.count - Self.maxMessages
        let toRemove = max(excess, Self.trimBatchSize)
        messages.removeFirst(min(toRemove, messages.count))
    }
}
```

The `trimBatchSize` is a minor optimization—removing messages one at a time is wasteful, so I remove 100 at once when the limit is hit. It's the kind of micro-optimization that probably doesn't matter but makes me feel clever.

## Act V: Tab Completion, or The Feature That Took Three Rewrites

Every IRC client needs tab completion. Type `@Al<TAB>` and it should complete to `@Alice: ` (with the colon if you're at the start of the line, because that's how you address someone on IRC, and yes, this matters to IRC users).

Version 1 didn't handle repeated tabs for cycling through candidates.

Version 2 handled cycling but broke when you typed anything after completing.

Version 3 worked but had a bug where the completion state wasn't preserved through SwiftUI's state updates:

```swift
// The hacky but functional solution
let savedCandidates = completionCandidates
let savedIndex = completionIndex

appState.inputText = newText  // This triggers onChange, which resets state

// Restore the state that was just reset
completionCandidates = savedCandidates
completionIndex = savedIndex
```

I'm not proud of it, but I'm not embarrassed either. Sometimes you have to work with the framework you have, not the framework you wish you had.

## Lessons Learned

1. **Scroll position is harder than it looks.** Every chat app has solved this problem, and every chat app has scars to show for it.

2. **Legacy protocols carry legacy baggage.** IRC's formatting codes are a window into a time when bandwidth was precious and consistency was for the weak.

3. **SwiftUI state management requires respect.** Fight the framework and you lose. Work with its grain, accept its quirks, and you might just ship something.

4. **The "simple" features take the longest.** Nobody spends weeks on the exotic features. It's the stuff everyone assumes will "just work" that consumes your calendar.

## The Result

I now have a chat window that:

- Scrolls smoothly
- Stays at the bottom when it should
- Lets you read history without interruption
- Renders decades-old formatting codes
- Completes nicks with a single tab
- Probably has three bugs I haven't found yet

It's not perfect, but it works. And in the grand tradition of IRC itself—a protocol that's survived 35+ years through sheer stubbornness—sometimes "works" is exactly good enough.

---

*cIRC is a native macOS/iOS IRC client built with Swift 6 and SwiftUI. If you've read this far, you're either very interested in chat interfaces or very good at procrastinating. Either way, I salute you.*
