---
title: "Building social feed algorithms using Graph DBs"
description: "What happens when a graph database enthusiast discovers a firehose of social data and has too much free time."
date: 2023-04-24
tags: ["graph-database", "algorithms", "bluesky", "social-media", "engineering"]
---

There's a peculiar kind of madness that seizes certain developers. It usually begins innocuously—perhaps you read about a new protocol, or notice an interesting API, or simply wonder "what if?" In my case, the madness struck when I was working at Memgraph and discovered that Bluesky, the nascent Twitter-alternative built by Jack Dorsey's wandering attention, had something Twitter had long since locked away: a firehose.

Not a trickle. Not a curated stream. A *firehose*—500 events per second of raw, unfiltered social chaos streaming through a WebSocket connection, free for anyone to drink from. Posts, likes, follows, reposts, unfollows—the entire nervous system of a social network, laid bare.

Reader, I drank deeply.

## Understanding the AT Protocol

For the uninitiated, Bluesky isn't just another social network with a nice logo and dreams of not becoming a cesspool. It's built on the AT Protocol (Authenticated Transfer Protocol), a genuinely fascinating piece of engineering that separates the *what* from the *where*. Your identity isn't tied to a server—it's cryptographically yours through Decentralized Identifiers (DIDs). Your data lives in a Personal Data Server (PDS) that you could, theoretically, host yourself. And everything flows through relays that aggregate the entire network's activity into that glorious firehose.

It's the kind of architecture that makes a graph database enthusiast's heart sing. Because what is a social network, really, but a graph? People connected to people, people connected to posts, posts connected to other posts through replies and quotes—it's relationships all the way down.

And I happened to work for a company that made a rather good graph database.

The project that emerged from this collision of circumstances was called **BlueJ** (originally "Home+"), and it became both my obsession and my education in the fine art of algorithmic feed generation.

## The problem with feeds

Here's the thing about chronological feeds: they're democratic to a fault. Every post from everyone you follow, in the order they posted it. Simple. Pure. And absolutely overwhelming once you follow more than fifty people.

The alternative—algorithmic feeds—has become something of a dirty word. I've all experienced the engagement-maximizing hellscape of platforms that show you whatever keeps you doomscrolling, regardless of whether it's good for your mental health or your relationships or your basic faith in humanity.

I wanted something in between. A feed that was *interesting* without being *manipulative*. A feed that surfaced quality content without optimizing for outrage. A feed that felt like it was working *for* me rather than *on* me.

So I did what any reasonable person would do: I wrote a bunch of Cypher queries and hoped for the best.

## The three-stream approach

The core insight of BlueJ's feed algorithm is embarrassingly simple: don't rely on a single signal. Instead, blend multiple sources of content using a weighted distribution.

The system runs three queries in parallel (because waiting for database queries sequentially is for people with more patience than deadlines):

```typescript
let queryResults = await parallelQueries(requesterDid, maxNodeId, {
    follow: { query: followQuery, limit: 300 },
    likedByFollow: { query: likedByFollowQuery, limit: 100 },
    community: { query: communityQuery, limit: 100 }
})
```

**Stream 1: Your Follows** — The bread and butter. Posts from people you've explicitly said you want to hear from. This gets the largest allocation (300 posts) because, well, you chose these people.

**Stream 2: Liked by Your Follows** — This is where it gets interesting. What are the people you trust finding valuable? It's a second-degree signal—social proof filtered through your own curation. Limited to 100 posts because you don't want your feed colonized by your friend's questionable taste in shitposts.

**Stream 3: Community Posts** — For users who've been assigned to community clusters (through graph analysis), posts from others in the same community. A way to discover people you *might* want to follow, based on the structure of the social graph rather than explicit choices.

## Scoring with time decay

Now here's where I'll confess to stealing shamelessly from the giants. The scoring function for ranking posts within each stream uses a decay formula that should look familiar to anyone who's studied Hacker News:

```cypher
WITH (ceil(likes) / ceil(1 + (hour_age * hour_age * hour_age * hour_age))) as score
```

That's `score = likes / (1 + hour_age^4)` for those keeping track at home.

The fourth power on the age denominator is *aggressive*. A post that's 4 hours old needs 256 times as many likes as a fresh post to achieve the same score. At 24 hours? 331,776 times as many likes. The formula doesn't just prefer fresh content—it *demands* it.

This prevents the "greatest hits" problem where the same viral posts linger at the top of your feed forever. New posts get their moment in the sun; if they don't earn engagement quickly, they gracefully fade away. It's Darwinian, but fair.

## Blending the streams

Here's a subtle problem: you've got three arrays of posts, each ranked by their own criteria. How do you merge them into a single feed without one source dominating?

My solution was a weighted round-robin algorithm that uses array length as the weight:

```typescript
export function weightedRoundRobin(...arrays: Element[][]): Element[] {
    const result: Element[] = [];
    const maxCount = Math.max(...arraysLengths);

    for (let i = 0; i < maxCount; i++) {
        for (let arrIndex = 0; arrIndex < arrays.length; arrIndex++) {
            const arr = arrays[arrIndex];
            if (i < arr.length) {
                result.push(arr[i]);
            }
        }
    }
    return result;
}
```

The elegance (if I may be permitted a moment of immodesty) is in the interleaving. With a 300/100/100 distribution, you get roughly 3 posts from follows, then 1 from liked-by-follows, then 1 from community, repeating. The relative proportions are preserved, but the content is mixed throughout the feed rather than segregated into sections.

It's like shuffling a deck of cards where some suits have more cards than others, but every hand still feels balanced.

## Detecting refresh requests

One of my favorite pieces of cleverness in the codebase (and I say this knowing full well that "favorite pieces of cleverness" is exactly what developers say before showing you their most over-engineered code) is the freshness detection system.

The problem: when a user pulls to refresh, they want to see *new* posts at the top. But when they're paginating through their feed, they want consistency—the same posts in the same order, just further down.

The solution: track the highest post ID each user has seen, and use the request parameters to infer intent.

```typescript
// A larger limit and no cursor indicates that this was not a new-post probe,
// but a full feed refresh, so we mark the highest Node.ID as the last seen post
if (limit >= 10 && cursor === undefined) {
    didLastSeen[requesterDid] = {
        maxNodeId: maxNodeId,
        timestamp: Date.now()
    }
}
```

When a refresh is detected, posts newer than the user's last seen ID bubble to the top:

```typescript
if (didLastSeen[requesterDid] !== undefined) {
    const newResults = results.filter((result) => result.id > didLastSeen[requesterDid].maxNodeId);
    const seenResults = results.filter((result) => result.id <= didLastSeen[requesterDid].maxNodeId);
    results = [...newResults, ...seenResults]
}
```

It's the kind of thing users never notice when it works, but would absolutely notice if it didn't. The best infrastructure is invisible.

## Ingesting the firehose

Of course, none of this matters if you can't get the data into your graph in the first place. The AT Protocol's firehose is a WebSocket stream of repository commit events encoded in CBOR (Concise Binary Object Representation) wrapped in CAR files (Content Addressed aRchives).

If that sentence made your eyes glaze over, you're not alone. Fortunately, the `@atproto` libraries handle most of the decoding. My job was simply to transform each event into a Cypher query and execute it against Memgraph:

```typescript
// For new posts
await this.executeQuery(
    "CREATE (post:Post {uri: $uri, cid: $cid, author: $author, text: $text, " +
    "createdAt: $createdAt, indexedAt: LocalDateTime()}) " +
    "MERGE (person:Person {did: $author}) " +
    "MERGE (person)-[:AUTHOR_OF {weight: 0}]->(post)",
    { uri, cid, author, text, createdAt }
)

// For follows
await this.executeQuery(
    "MERGE (p1:Person {did: $authorDid}) " +
    "MERGE (p2:Person {did: $subjectDid}) " +
    "MERGE (p1)-[:FOLLOW {weight: 2, uri: $uri}]->(p2)",
    { authorDid, subjectDid, uri }
)
```

The `MERGE` pattern is crucial here—it creates the node if it doesn't exist, or matches the existing one if it does. This handles the out-of-order nature of the firehose gracefully. Someone might like a post before I've seen the post itself; the MERGE ensures I create placeholder nodes that get enriched later.

For resilience, failed queries go into a retry queue processed every 5 seconds:

```typescript
private queryQueue: RetryableQuery[];

async executeQuery(query: string, params: object, retryCount: number = 10) {
    try {
        results = await session.run(query, params);
    } catch (error) {
        if (this.isRetryableError(error) && retryCount > 0) {
            this.queryQueue.push({ query, params, retryCount: retryCount - 1 })
        }
    }
}
```

It's not sophisticated, but it's remarkably effective at handling the occasional database hiccup without losing data.

## Real-time visualization

Now, building a custom feed generator is all well and good, but there's something deeply unsatisfying about infrastructure that only manifests as a slightly better list of posts. I wanted to *see* the graph. To watch it grow. To witness the social network *breathe*.

So I built a real-time visualization layer.

The stack is deceptively simple: React on the frontend with `react-force-graph` for the 3D/2D rendering, Socket.io for real-time updates, and Memgraph triggers firing HTTP callbacks whenever nodes or edges change.

The visualization backend subscribes to a user's network and streams updates:

```javascript
socket.on('interest', async (handle) => {
    // Find the user and their follows
    // Return initial graph state
    // Subscribe to updates for all relevant DIDs
})
```

Memgraph triggers detect changes and push them through a C++ query module to the visualization service, which broadcasts to interested clients. The result is hypnotic—nodes appearing and connecting, edges forming and dissolving, a living representation of human connection rendered in WebGL.

You can watch someone gain a follower in real-time. See a post accumulate likes as little edges snake toward it. Witness the graph reorganize itself as the force-directed layout algorithm seeks equilibrium.

It is, I must admit, completely impractical for any real purpose. But it's *beautiful*, and sometimes that's enough.

## What I learned

Building BlueJ taught me several things:

**First**: social graphs are *hard*. Not computationally hard—Memgraph handles millions of nodes without breaking a sweat—but *semantically* hard. What does a "follow" mean? How much weight should a "like" carry versus a "reply"? These are fundamentally human questions with no correct technical answer.

**Second**: the AT Protocol is genuinely interesting. The separation of identity from hosting, the cryptographic verification of content, the openness of the firehose—it represents a real philosophical departure from the walled gardens we've grown accustomed to. Whether Bluesky itself succeeds matters less than whether the protocol inspires alternatives.

**Third**: algorithmic feeds don't have to be evil. The engagement-maximizing nightmares we've experienced aren't an inevitable consequence of algorithmic curation—they're a consequence of *optimizing for the wrong things*. A feed that optimizes for user satisfaction rather than engagement metrics can be genuinely helpful.

**Fourth**: there's immense joy in building something that scratches your own itch. BlueJ was never going to compete with the official Bluesky apps. It was never going to scale to millions of users. But it was *mine*, built to my tastes, solving my problems. In an era of platforms, there's something radical about that.

## Current state

I no longer work at Memgraph, and BlueJ is no longer actively maintained. The codebase lives on as a template, a proof of concept, a testament to what's possible when you have a graph database, a firehose, and insufficient adult supervision.

Bluesky, meanwhile, has exploded. What was a million users when I built this is now 40 million and growing. The firehose is busier than ever. The protocol continues to evolve.

If you're the kind of person who reads technical blog posts about graph algorithms and social media protocols—and if you've made it this far, you clearly are—I encourage you to explore the AT Protocol yourself. The water's fine. The firehose is free. And there's something deeply satisfying about building your own corner of the social web.

Just don't blame me when you find yourself at 2 AM, watching nodes connect in a force-directed graph, muttering about Cypher query optimization.

I've been there.

---

*The BlueJ codebase is available for the curious and the foolhardy. The author accepts no responsibility for any obsessions, rabbit holes, or sudden urges to build custom feed generators that may result from examining the code.*

*Originally built at Memgraph. Current state: archived, but fondly remembered.*
