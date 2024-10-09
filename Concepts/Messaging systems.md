## Overview
A messaging system is used to enable following:
- Buffering: Decoupling systems in terms of load, so if a system gets a lot of load, a messaging system can act as a buffer for the requests so the system can work on its own pace.
- Loose coupling and guarantee of delivery: Introduces loose coupling between producer services and consumer services. A producer can produce a message on a messaging system with a full guaratee that it will eventually be delivered to the consumers.

## Types of messaging systems
- Messaging queue: works as a queue, each message is consumed by one consumer.
- Publish-susbcribe: a single message can be consumed by multiple subscribers. The system that stores the message is generally called broker.
