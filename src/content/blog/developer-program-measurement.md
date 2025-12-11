---
title: "Measuring what matters in DevRel"
description: "Developer Relations has a metrics problem. Not a shortage of them—we've got GitHub stars and Discord members galore. The problem is none of them actually mean anything."
date: 2025-12-11
tags: ["devrel", "metrics", "developer-experience", "strategy"]
---

*"Games are won by players who focus on the playing field—not by those whose eyes are glued to the scoreboard."*
— Warren Buffett, who never had to justify a DevRel budget to a board of directors

## The conference industrial complex

I've seen it countless times across the industry—in teams I've worked with, in programs I've observed, and most recently in a team a good friend joined as the new leader. The strategy is seductively simple: fly somewhere photogenic, give a talk, collect business cards, post photos to LinkedIn with captions about "incredible energy" and "amazing community." Rinse. Repeat. Call it success.

This is **developer tourism**—the art of being everywhere while accomplishing approximately nothing measurable. It's the DevRel equivalent of a politician kissing babies: photogenic, feels important, produces zero lasting change.

The problem isn't that conferences don't work. Some do. The problem is we have no idea *which* ones work, because we've been measuring the wrong things entirely.

---

## The metrics graveyard

Developer Relations has a metrics problem. Not a shortage of them—Lord knows we've got GitHub stars, Discord members, and enough UTM-tagged links to wallpaper a conference hall. The problem is that none of them actually *mean* anything.

According to the [State of Developer Relations 2024](https://www.stateofdeveloperrelations.com/2024devrelreport), proving business impact jumped from 7th to 2nd place among DevRel challenges. A full 61% of practitioners struggle to prove their influence. We've built an entire industry on vibes, and the vibes are getting nervous.

The metrics we collect are what Mary Thengvall calls "[easily gamed vanity metrics](https://develocity.io/measuring-devrel-success-effective-metrics-for-impact/)." GitHub stars? Bots love those. Discord members? Half are lurkers who joined for an airdrop. Event attendance? Congratulations, you attracted people to free pizza.

None of these answer the question that actually matters: **Did anyone ship code afterward?**

---

## Enter Electric Capital

While DevRel teams have been counting Discord emojis, a quiet revolution has been happening in data infrastructure. [Electric Capital](https://www.electriccapital.com/) has been tracking something radical: actual developer activity.

Their methodology is elegantly simple. Analyze public GitHub commits. Map developers to ecosystems. Count who's building what, where. Their [2024 Developer Report](https://www.developerreport.com/reports/devs/2024) analyzed 902 million code commits across 1.7 million repositories—the kind of dataset that makes vanity metrics look like finger paintings.

What Electric Capital provides is something DevRel has desperately needed: **a ground truth**. Not self-reported surveys. Not signup forms. Not aspirational Discord channels. Actual, verifiable, "this person wrote code and pushed it to GitHub" evidence.

The methodology matters:
- **Monthly Active Developers (MADs)**: Anyone who committed code in a rolling 28-day window
- **Developer tenure**: Time since first commit to *any* blockchain (not just yours)
- **Activity classification**: Full-time (10+ days/month), part-time (regular but not daily), one-time contributors

This is the difference between "we have a vibrant community" and "we have 847 developers who committed code last month, 23% of whom have been building in crypto for over two years."

One of these sentences survives a board meeting. The other does not.

---

## From tourism to traction

The dashboard we built does something previously impossible: it connects developer relations activities to actual outcomes. Not in the abstract. Not with attribution models so complex they require a PhD to understand. Just: did this person, after attending this event, write code?

### The participation funnel

Every event now produces a conversion funnel:

```
Total Participants → Resolved to GitHub → New Developers → Activated (7/14/28/60 days)
```

"Resolved to GitHub" is the key step. We take your messy CSV of hackathon registrants—with their inconsistent column names and email addresses that may or may not match anything—and match them to canonical developers in the Electric Capital database.

The resolution is obsessive. We scan every column for GitHub URLs. We extract usernames from repository URLs that no longer exist. We match emails against developer profiles. We check repository contributor lists. We're like digital archaeologists, except instead of pottery shards, we're piecing together who actually showed up and whether they stuck around.

Then comes the magic number: **activation rate**. What percentage of new developers who attended your hackathon were still committing code 28 days later? 60 days?

This is the metric that separates developer growth from developer tourism. Anyone can attract a crowd. The question is whether they become builders.

---

## The insights that actually matter

### Churn risk: your crystal ball for attrition

We built a churn risk scoring system that identifies developers about to go silent. It's slightly creepy and extremely useful.

The model looks at:
- Last commit date (longer ago = higher risk)
- Declining activity trends (falling off vs. steady)
- Commit patterns (small, infrequent commits = disengaging)

Why does this matter? Because **retention is cheaper than acquisition**. A developer who's shipped 100 commits and is losing steam is worth an email, a DM, a "hey, what are you working on?" That person already knows your platform. They've already climbed the learning curve. Letting them drift away is a failure of relationship, not marketing.

The dashboard flags these at-risk developers before they disappear. It's developer relations as customer success—proactive, data-driven, and considerably less expensive than flying to another conference.

### Retention cohorts: the true north

The 30/60/90-day retention cohort analysis answers a question most DevRel teams can't: **Is our onboarding getting better?**

Compare the September cohort's 30-day retention to October's. If October is higher, something you did (or fixed) worked. If it's lower, something broke. This is A/B testing for developer experience, except the test group is everyone.

The color coding is merciless. Green means ≥50% retention (you're doing something right). Yellow means 30-49% (room for improvement). Red means you might want to examine your first-run experience with fresh eyes and perhaps a stiff drink.

### The multichain signal

Here's a metric I didn't expect to care about: the exclusive vs. multichain developer split.

**Exclusive developers** only contribute to your ecosystem. **Multichain developers** hedge their bets across multiple blockchains.

Both have value, but they tell different stories. A high exclusive ratio suggests you've built something developers commit to (literally and figuratively). A high multichain ratio means you're attracting experienced crypto developers who are keeping options open.

Neither is inherently better. But if you're measuring "developer mindshare," this is what mindshare actually looks like in the data.

### The package pipeline: from code to chain

The full developer journey isn't commit → done. In Web3, it's:

```
Developer → Code Commit → Package Deployment → On-chain Transactions → Actual Usage
```

We track all of it. The dashboard links 278,000+ Move packages to their GitHub repositories, then to their developers, then to their on-chain transaction counts. You can now answer: "Which hackathon participants have packages generating real usage?"

This is the developer-to-revenue pipeline that DevRel teams talk about in theory but rarely measure in practice. Turns out, with enough data infrastructure, you can actually close the loop.

---

## The uncomfortable truths

### Employee filtering: the awkward conversation

When you toggle "Exclude Employees," you discover what community growth looks like without your company's own developers padding the numbers.

This is... informative.

In some ecosystems, employee commits account for 50%+ of activity. That's not a community—that's a company with a Discord server. The dashboard makes this visible, which is uncomfortable but necessary. You can't improve what you won't measure.

### Geography: where is your community, really?

The geographic distribution data ends the "global community" fantasy. You can see exactly which countries and cities your developers come from, and it's often not where you've been speaking.

Maybe you should stop flying to San Francisco and start engaging developers in India, which has become the [second-largest source of crypto developers](https://www.chaincatcher.com/en/article/2156880) globally. Or maybe your community really is concentrated in three cities, and you should double down there instead of spreading thin.

Either way, now you know.

### The heatmap: when they actually work

The commit activity heatmap shows when developers actually write code. Spoiler: it's not 2 AM. The romanticized image of the nocturnal hacker is largely fiction. Most commits happen during business hours in developer-heavy time zones.

This matters for scheduling: launches, announcements, support coverage, office hours. The data tells you when your developers are at their keyboards. Everything else is guessing.

---

## Building an impact-focused strategy

So what changes when you measure what matters?

### 1. Pre-event benchmarking

Before your next hackathon, pull the baseline metrics. How many MADs do you have? What's your retention rate? What's your new developer acquisition trend?

After the event, measure the delta. Did MADs increase? Did retention improve? Did you acquire new developers who stuck around?

If you can't show movement, the event didn't work—regardless of how good the swag was.

### 2. Double down on activation

The participation funnel reveals where developers drop off. If you're getting signups but not activations, your onboarding is broken. If you're getting activations but not 28-day retention, your platform has a cliff somewhere.

Fix the bottleneck, not the symptom.

### 3. Prioritize retention interventions

The churn risk score isn't just analytics—it's a to-do list. Reach out to at-risk developers before they disappear. A personal email to someone who's shipped 50 commits is worth more than a generic newsletter to 10,000 addresses.

### 4. Measure outcomes, not activities

Stop counting events attended. Start counting:
- New developers acquired per event
- Activation rates per event
- Cost per activated developer

Now you can compare: Was the $50K hackathon more effective than the $5K community meetup series? With actual data, you can make actual decisions.

---

## The path forward

The [DevRel industry is stabilizing](https://www.devrel.agency/post/announcing-the-11th-annual-state-of-developer-relations-report)—median compensation is $193K, job security is steady, and 78% of professionals now use AI in their workflows. The teams that survive and thrive will be the ones that can answer the accountability question.

Electric Capital provided the data infrastructure. This dashboard turns it into actionable intelligence. The rest is execution.

For me personally, this approach has been transformative. It's helped me convince leadership and horizontal stakeholders to trust a data-driven strategy. More importantly, it's given me the clarity to identify which events were actually working for us, which weren't, and what specific changes we needed to make to drive real success.

I still believe in the power of being present—of handshakes and whiteboard sessions and the serendipity of hallway conversations. But presence without measurement is just expensive travel.

The next time someone asks "What's the ROI of that conference?", you should have an answer beyond "the energy was amazing."

Because the energy doesn't show up in the investor deck. Developer activation does.

---

## Technical note

This analysis is powered by data from [Electric Capital's Developer Report](https://www.developerreport.com/), enriched with GitHub profile data, Sui blockchain package analytics, and custom retention modeling. The dashboard tracks three ecosystems (Sui, Walrus, Sui Name Service) with daily data updates.

The methodology:
- **MADs**: 28-day rolling window of commit activity
- **Retention cohorts**: Monthly cohort tracking with 30/60/90-day windows
- **Event attribution**: Temporal matching (14 days before to 28 days after event)
- **Package linking**: Hybrid MVR + Blockberry data with confidence scoring

For the technically curious, the full methodology is documented in the repository. For the strategically curious, the answer is simpler: we finally measured what conferences couldn't tell us.

---

*Want to stop guessing and start measuring? The dashboard is built for the Sui ecosystem, but the methodology applies anywhere developers ship code. The era of DevRel vibes is over. The era of DevRel data has begun.*

---

## Sources

- [Electric Capital Developer Report 2024](https://www.developerreport.com/reports/devs/2024)
- [State of Developer Relations 2024](https://www.stateofdeveloperrelations.com/2024devrelreport)
- [Measuring DevRel Success: Effective Metrics for Impact](https://develocity.io/measuring-devrel-success-effective-metrics-for-impact/)
- [DevRel Metrics and Why They Matter](https://seanfalconer.medium.com/devrel-metrics-and-why-they-matter-224563a4aa2d)
- [Measuring the Impact of Your Developer Relations Team - OpenView](https://openviewpartners.com/blog/measuring-the-impact-of-your-developer-relations-team/)
- [Electric Capital 2024 Developer Report Analysis - ChainCatcher](https://www.chaincatcher.com/en/article/2156880)
- [How Hackathons Generate ROI - Hackathon.com](https://corporate.hackathon.com/articles/how-hackathons-generate-roi-for-products-and-services)
