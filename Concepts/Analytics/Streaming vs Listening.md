### Listener (Consumer/Listener Service)

* **Purpose:** Just consumes messages from a queue or topic (e.g. Kafka, RabbitMQ) and processes them.
* **When to use:**

  * Processing is relatively simple (store in DB, send email, update cache).
  * Order is not super critical (unless you do some ordering magic in consumer group).
  * Latency requirements are *near real-time*, not strictly low-latency.
  * You want independent scaling — add more consumers for higher throughput.
  * No complex state is required across messages.

**Example:** User sign-up events → send welcome email, log analytics event → simple consumer service works great.

---

### Stream Processor (Streaming Engine: Kafka Streams, Apache Flink, Spark Streaming)

* **Purpose:** *Processes streams of data in a stateful way* — windowing, joins, aggregations, running counts, deduplication, complex event processing.
* **When to use:**

  * You need **stateful processing** — e.g. aggregate views, top-k, counting unique users per hour, detecting patterns across events.
  * You care about **event-time** semantics (processing late-arriving data correctly).
  * Processing must scale horizontally and handle distributed state.
  * You want **windowed operations** — per 10-min window, rolling average, etc.
  * You need **exactly-once** or at-least-once guarantees in complex pipelines.
  * Low-latency is critical, but also correctness and state handling.

**Example:** Real-time fraud detection, showing top videos per hour (you just asked about this in another thread 😉), updating leaderboards, monitoring metrics in real-time dashboards.

---

### TL;DR for interview

👉 **Listener:** "Good for simple, stateless consumption of events — useful when processing logic is basic and you just want to fan out work."

👉 **Stream Processor:** "Used when I need to manage state across events — for aggregations, windowing, joins, and complex pipelines with correctness guarantees."
