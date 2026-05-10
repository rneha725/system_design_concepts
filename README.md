<details>
<summary>Approaching a system design problem</summary>
  
## The Model: **Find the Promise**

Every system exists to make one guarantee to its users.

> Ticket booking: *"The seat you paid for is yours."*
> Social feed: *"You see what the people you follow have posted."*
> URL shortener: *"This link always takes you to the right page."*

Write that sentence first. Everything else is derived from it.

---

## FRs = What the system must do to keep the promise

Ask: **what capabilities are necessary and sufficient for this promise to hold?**

Not "what features could this system have" — that's how you get bloated requirement lists. Specifically: what must be true for the promise to not be broken?

For ticket booking: inventory must be accurate, payment must be atomic with reservation, booking must be retrievable. Those are your FRs. Nothing else is load-bearing.

---

## NFRs = The conditions under which the promise must hold

The promise doesn't exist in a vacuum. It must hold:

- **under load** → scale numbers
- **within time** → latency targets  
- **despite failure** → availability, durability
- **under contention** → consistency model

NFRs are just the stress conditions on the promise. If the promise is *"the seat you paid for is yours"*, then the NFR question is: does that hold when 50,000 people hit checkout simultaneously? Does it hold if the payment service goes down mid-transaction? Does it hold if a DB node dies?

---

## Why This Is Better

The checklist model (ACTOR → FLOW → STRESS) tells you *where to look*. This model tells you *what you're looking for*.

The promise gives you a filter. When you're mid-interview and unsure if something is a requirement, you ask: **does this affect whether the promise holds?** If yes, it's a requirement. If no, cut it.

It also means your requirements justify your architecture automatically — because every design decision is just "this is how we keep the promise under this condition."

```
Promise breaks under contention
  → need strong consistency for inventory
  → pessimistic locking

Promise breaks if payment succeeds but booking fails
  → need atomicity across payment + reservation
  → authorize-capture pattern + idempotent confirmation

Promise breaks at 500K reads/sec
  → need pre-computed cache
  → fan-out on write
```

The architecture is just the promise, defended.

</details>
<details>
  <summary>Examples of FRs and NFRs</summary>

  ## The Promise → FRs → NFRs, With Many Systems

---

### Ticket Booking
**Promise:** *The seat you paid for is yours.*

**FRs** (what must be true for the promise to hold):
- Users can search and browse events
- Users can see a real-time seat map
- A seat can be reserved temporarily while payment is processed
- Payment and reservation are atomic — one cannot succeed without the other
- A booking is retrievable after confirmation
- A released/expired reservation returns the seat to inventory

**NFRs** (conditions under which the promise must hold):
- Inventory must be strongly consistent — eventual is not acceptable (two people cannot own the same seat)
- Seat reservation within 500ms p99
- System must handle 50K concurrent users on hot event drops
- Booking records must be durable — survive any single node failure
- Payment service downtime must not cause silent data loss

---

### Social Media Feed
**Promise:** *You see what the people you follow have posted.*

**FRs:**
- Users can follow and unfollow other users
- Posts are created and attached to an author
- A user's feed returns posts from their followed set, ordered by recency or ranking
- New posts appear in followers' feeds within some bounded time window

**NFRs:**
- Feed load under 300ms p99 (users leave if it's slow)
- Eventual consistency is acceptable — seeing a post 2 seconds late is fine
- Write throughput: handle celebrity accounts with 10M+ followers without fan-out blowing up
- 100M DAU, reads vastly outnumber writes — read path must be independently scalable
- Feed data can be stale by seconds, not minutes

---

### Ride-Sharing (Uber)
**Promise:** *A driver will come to you.*

**FRs:**
- Riders can request a ride from their current location
- System matches riders to nearby available drivers
- Driver location is tracked in real time
- Rider can see driver ETA and live position
- Trip is recorded with start/end location and fare
- Rider and driver can cancel within constraints

**NFRs:**
- Driver location updates must be near real-time (< 5 second staleness)
- Matching must complete within 2–3 seconds of request
- Location data is write-heavy (millions of drivers pinging every few seconds) — write path must scale horizontally
- Availability over consistency for matching — a slightly stale driver position is fine, a failed match is not
- Geo-queries (find drivers within X km) must be fast — requires spatial indexing

---

### Web Crawler
**Promise:** *We have a fresh, complete index of the web.*

**FRs:**
- System can fetch and parse a URL
- Discovered links are enqueued for future crawling
- Already-crawled URLs are not re-crawled unnecessarily
- Page content is stored and made available to the indexing pipeline
- Crawl respects robots.txt

**NFRs:**
- Must scale to billions of URLs — single machine is not an option
- Freshness: high-priority pages (news) re-crawled within hours; static pages within weeks
- Politeness: no single domain gets hammered — rate limiting per domain
- Deduplication must be fast at billion-URL scale — Bloom filter or consistent hashing, not a full DB lookup
- Fault tolerant: a crawler node dying must not lose its queue

---

### Rate Limiter
**Promise:** *No client can exceed their allowed request quota.*

**FRs:**
- For any incoming request, system decides allow or reject based on client identity + current usage
- Usage counters are tracked per client per time window
- Rejected requests receive a meaningful response (429 + retry-after header)
- Limits are configurable per client tier

**NFRs:**
- Decision must be made in < 5ms — it sits in the hot path of every request
- Must be accurate under concurrent requests from the same client hitting different nodes
- Consistency trade-off: slight over-counting is acceptable; significant under-counting is not
- Must scale to millions of distinct client IDs
- Redis counter TTL drift or clock skew between nodes must not cause correctness failures

---

## The Pattern Across All of Them

FRs always answer: **what actions must the system support to deliver the promise?**

NFRs always answer: **what are the hardest conditions under which the promise must still hold?**

The NFRs are where the architecture lives. Every hard NFR is a forcing function:

| Hard NFR | Forces |
|---|---|
| Strong consistency under contention | Pessimistic locking, single-writer pattern |
| < 5ms decision latency | In-memory store, no DB in hot path |
| 10M-follower fan-out | Read-time merge, not write-time fan-out |
| Billions of URLs, dedup fast | Probabilistic data structures |
| Location updates every 5s × 1M drivers | Write-optimized store, time-series or geospatial index |

If your NFRs aren't forcing architectural decisions, they're not real NFRs — they're decoration.
</details>

