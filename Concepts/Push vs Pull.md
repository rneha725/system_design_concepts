### Push Model

* The **producer (sender)** *actively sends* data to the consumer (receiver), without the consumer asking for it each time.
* Flow is **sender-driven**.

**Pros:**
✅ Lower latency — data arrives as soon as it is available.
✅ Efficient when producer knows when data is ready.
✅ Good for real-time systems (monitoring alerts, live updates).

**Cons:**
❌ Consumer can be overwhelmed if too much data is pushed (backpressure handling required).
❌ Harder to control flow at consumer side.

**Example:**

* Email notifications
* Monitoring agents pushing metrics to a monitoring system
* Webhooks

---

### Pull Model

* The **consumer (receiver)** *actively requests* data from the producer.
* Flow is **consumer-driven**.

**Pros:**
✅ Consumer controls the pace — avoids getting overwhelmed.
✅ Easier retry logic (just pull again).
✅ Simpler to implement in many cases (HTTP API, periodic polling).

**Cons:**
❌ Higher latency (consumer pulls periodically, may miss updates in between).
❌ Wasteful if polling frequently with no new data.

**Example:**

* REST APIs — client pulls data when it wants.
* Dashboard querying a database.
* Monitoring system pulling metrics from exposed endpoints (Prometheus style).

---

### Interview-ready TL;DR

**Push** — *sender-driven, lower latency, great for real-time but needs flow control.*
**Pull** — *consumer-driven, safer for consumer, may add latency or waste cycles.*

---

### Pro tip:

In **monitoring systems**, you often choose between:

* **Push**: agents push metrics (Prometheus PushGateway, some cloud monitoring systems).
* **Pull**: central server scrapes endpoints (Prometheus default model — `/metrics` endpoint).

In **event-driven architectures**:

* Kafka **pulls** — consumers pull from the broker at their own pace.
* Webhooks **push** — source system fires off HTTP requests to notify subscribers.

---

### When asked in system design:

👉 *If low latency and event-driven, mention push.*
👉 *If scalable, controlled processing is key, mention pull.*
👉 *Sometimes hybrid: push to queue → pull by consumers at controlled pace (Kafka, SQS).*


| **Component / Context**                | **Push or Pull?**          | **Why?**                                                                          |
| -------------------------------------- | -------------------------- | --------------------------------------------------------------------------------- |
| **Monitoring (Prometheus, Grafana)**   | **Pull** (default)         | Central server controls scrape, avoids overload, better for large fleet.          |
| **Monitoring (CloudWatch, Datadog)**   | **Push**                   | Agents push to centralized system, fits SaaS model.                               |
| **Alerting system**                    | **Push**                   | Fast reaction needed — e.g. PagerDuty via webhook.                                |
| **Metrics collection (IoT devices)**   | **Push**                   | Devices may not be reachable to pull from — push to cloud works better.           |
| **Database querying**                  | **Pull**                   | Client initiates query when needed — avoids unnecessary DB load.                  |
| **REST APIs**                          | **Pull**                   | Client controls request timing and load.                                          |
| **Webhooks**                           | **Push**                   | Source sends updates to subscribers — event-driven.                               |
| **Kafka consumer**                     | **Pull**                   | Consumer controls rate, enables retries, batching.                                |
| **Kafka producer → broker**            | **Push**                   | Producer pushes data into Kafka.                                                  |
| **Email notifications**                | **Push**                   | Sender initiates delivery.                                                        |
| **Mobile push notifications**          | **Push**                   | Server initiates notification to device.                                          |
| **Client polling for updates**         | **Pull**                   | Simple to implement but can waste cycles.                                         |
| **Live chat / WebSocket**              | **Push** (with open conn.) | Server pushes updates in real-time over persistent connection.                    |
| **Stock price updates on dashboard**   | **Push** via WebSocket     | Low-latency updates needed.                                                       |
| **Video streaming (Netflix, YouTube)** | **Pull**                   | Client requests data at its own pace — helps with buffering and adaptive bitrate. |

