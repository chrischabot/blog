---
title: "You Don't Know What You Know (Until You're Asked)"
description: "Building Interviewer: a multi-agent AI app that conducts podcast-style interviews and transforms conversations into polished essays."
date: 2024-12-10
tags: ["ai", "swift", "agents", "projects"]
---

# You Don't Know What You Know (Until You're Asked)

There is a peculiar tragedy that befalls anyone who has accumulated expertise: they forget what they know. Not in the amnesiac sense, but in a subtler, more insidious way. Knowledge becomes so deeply embedded that it hides in plain sight, like furniture you've stopped seeing because it's always been there.

I've spent years thinking about things—AI, software architecture, the peculiar habits of early-stage startups—and I suspect I know quite a bit about them. But ask me to write it down and I'll stare at a blinking cursor for an embarrassingly long time, eventually producing something that sounds vaguely like every other blog post on the internet.

The problem isn't a lack of ideas. The problem is that nobody is asking me the right questions.

<!-- Image placeholder: The Interviewer app home screen -->

## The Curious Case of the Missing Interlocutor

Good interviews are a kind of magic. The right question at the right moment can unlock insights you didn't know you had. Skilled interviewers—the Terry Gross types—don't just collect information. They *excavate* it. They probe contradictions, follow tangents, circle back to half-finished thoughts that the speaker themselves had forgotten.

But good interviewers are in short supply, and most of us don't have one on retainer.

Enter **Interviewer**: a native Swift app that conducts podcast-style interviews on any topic you choose, tracks what you say, researches threads in real-time, and transforms the conversation into a blog-ready essay. Fourteen minutes of talking. One piece of writing that sounds like you—because it *is* you, just polished.

The tagline says it all: *"You don't know what you know, until you are asked."*

## How It Actually Works

The experience is deceptively simple. You tell the app what you want to talk about—perhaps "the future of AI-native applications" or "why most code review is theater"—and it generates an interview plan. Not a rigid script, but a research-driven rubric with backbone questions (the ones you must hit) and follow-up tangents (the whimsical detours where insight often hides).

Then you talk. Just... talk. The app listens, responds with voice, and gently steers the conversation. A timer ticks in the corner, but there's built-in flexibility for the kind of meandering that produces gold.

Behind the scenes, something considerably more sophisticated is happening.

## Seven Agents, One Symphony

The real engineering achievement here isn't the voice interface—though that's elegant enough. It's the orchestration of seven specialized AI agents working in concert, like a well-rehearsed ensemble where each player knows exactly when to step forward and when to fade back.

**The Planner** drafts the interview before you begin, encoding hypotheses to probe and angles to explore. **The Note-Taker** runs continuously during the conversation, tracking key ideas, stories, claims, gaps, even contradictions in your thinking. **The Researcher** conducts web searches when you mention something unfamiliar, arming the interviewer with context.

**The Orchestrator** is perhaps the most interesting. Every few seconds, it surveys the notes, the research findings, the elapsed time, and the original plan—then decides what to ask next. Should we go deeper on this tangent, or pull back to cover something important? The Orchestrator makes that call.

After the interview, **Analysis** extracts themes, tensions, and quotable lines. Then **The Writer** produces the essay itself—first-person, in your voice, not a sanitized summary but something that sounds like you wrote it on your best day.

And if you have more to say, there's a **Follow-Up Agent** that analyzes your previous session and suggests topics for a six-minute continuation. Knowledge compounds.

<!-- Image placeholder: Choosing follow-up topics -->

## The Voice of the Writer (Which Is Your Voice)

Here's where most AI writing tools go wrong: they produce prose that sounds like AI wrote it. Generic. Bloodless. Full of phrases like "it's important to note" and "in today's fast-paced world."

Interviewer takes a different approach. The Writer agent is a ghostwriter, not a journalist. It extracts your natural patterns and cadence from the transcript. If you said "gnarly problem," it writes "gnarly problem"—not "complex challenge." If you build arguments in a particular way, rambling toward the point before snapping it into focus, the essay preserves that rhythm.

The output is elegant and well-structured, but unmistakably yours. The goal is for you to read it and think: *Yes, that's exactly what I meant—and exactly how I would have said it if I'd had time to write it properly.*

## No Backend, No Excuses

The entire system runs natively on your Mac or iPhone. No server to deploy, no backend to maintain. Your data stays on your device (stored in SwiftData, API key secured in Keychain). The only external communication is with OpenAI's APIs.

This simplicity is a feature, not a limitation. Lower latency, because there's no round-trip to a middleman server. Better privacy, because your half-formed thoughts about competitive strategy or your unconventional opinions about tab versus spaces never touch someone else's server.

<!-- Image placeholder: Live interview in progress -->

## The Technology Under the Hood

For the technically curious: this is Swift 6 running on SwiftUI with Apple's new Liquid Glass design language. The voice pipeline uses AVAudioEngine for capture and playback, with multi-layer echo cancellation so the AI doesn't interview itself. The agents are Swift actors calling OpenAI's Chat Completions API with structured outputs—meaning they return guaranteed JSON that the app can reason about.

The Realtime API handles voice-to-voice conversation with sub-500ms latency. Server-side voice activity detection means the AI knows when you've finished a thought without awkward pauses or interruptions.

158 tests verify the agent orchestration works correctly. This is not a demo project.

## Why This Matters

Every knowledge worker I know has the same complaint: *I don't have time to write.* But the dirty secret is that writing isn't the bottleneck—thinking is. Or rather, the specific kind of thinking that happens when you're forced to articulate ideas in complete sentences, with someone asking "but what do you mean by that?" and "can you give me an example?"

Interviewer doesn't write for you. It helps you think out loud, captures that thinking, and shapes it into something others can read. The essay that emerges isn't AI-generated content. It's your content, with AI as the editorial assistant you always wished you had.

Fourteen minutes of talking. One blog post that sounds like you.

That's the deal.

---

*Interviewer is a native Swift app for macOS 26 Tahoe and iOS 26. It requires an OpenAI API key and a willingness to discover what you actually think about things.*
