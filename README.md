# Approaching a system design question

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
