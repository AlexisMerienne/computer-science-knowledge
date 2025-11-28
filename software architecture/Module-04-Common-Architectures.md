# Module 4 — Common Software Architectures

Objective: Explore popular software architectures, how to apply them to C++ systems, and how to choose the right architecture based on constraints, team setup, and operational requirements.

Navigation: [Prev: Module 3](Module-03-Design-Patterns.md) • [Next: Module 5](Module-05-Dependency-Modularity.md)


## 4.1 Layered Architecture

Layered architecture divides a system into layers (presentation, application, domain, infrastructure). Each layer has a specific responsibility and communicates with adjacent layers.

Real-world use — embedded device firmware:

- Presentation: optional management API (web UI or CLI) to configure device settings
- Application: orchestration of features (power control, sensor sampling)
- Domain: business logic (control algorithms) implemented for real-time constraints
- Infrastructure: hardware drivers, networking stack

Why layering matters for C++:

- Clear boundaries reduce coupling between hardware drivers (often low-level C/C++) and higher-level algorithms (C++), which improves portability and testability (mock drivers for unit tests).
- Layering helps isolate real-time code from non-real-time tasks, allowing different build and test regimes.


### Diagram (mermaid)

```mermaid
flowchart LR
  UI[Presentation]
  App[Application Services]
  Domain[Domain Objects]
  Infra[Infrastructure / Persistence]
  UI --> App --> Domain --> Infra
```


## 4.2 MVC (Model-View-Controller)

MVC splits UI apps into Model (state), View (render), and Controller (input): it prevents UI and business logic from becoming entangled.

Real-world use — desktop CAD tools / editors:

- Model: document data structures (geometries, settings)
- View: interactive canvas, rendering engine (often GPU-accelerated)
- Controller: keyboard/mouse handlers, editor commands

C++ specifics:

- Many GUI frameworks (Qt) provide signal/slot mechanisms that map well to Observer in MVC.
- Avoid stuffing business logic inside controllers to keep code testable—push core operations into Model classes.



## 4.3 Microservices (introduction and implications for C++)

Microservices split systems into independently deployable services, each owning its data and API. They work well for large organizations or systems that require independent scaling and ownership.

Real-world use — media-transcoding platform:

- Separate services for upload ingestion, transcoding (CPU/GPU-heavy), metadata extraction, and delivery.
- Transcoding services can be C++ processes optimized for performance, while user-facing services may be implemented in other languages.

Important C++ considerations:

- Interoperability: use language-neutral IPC (gRPC/Protobuf, HTTP/JSON) with carefully versioned contracts.
- Binary size and cold start: C++ microservices often have small startup, but static linking and large runtime libraries can increase size; consider container image size and resource constraints.
- Packaging and cross-platform CI: ensure consistent toolchains for reproducible builds.
- Observability: ensure structured logging, metrics, and traces are available so compiled C++ services can be diagnosed in production.

Trade-offs: microservices increase operational complexity (Kubernetes, service discovery, deployment automation) and are not always the best choice for small teams or simple domains.


## 4.4 Event-Driven Architecture

Event-driven systems use streams or messages to decouple producers from consumers. This suits high-throughput, asynchronous pipelines (telemetry ingestion, real-time analytics).

Real-world use — telemetry processing pipeline:

- Producers: edge devices publish events
- Broker: Kafka handles large ingest and partitions/replicates streams
- Consumers: C++ services reading partitions, performing enrichment, and writing outputs

Important patterns and C++ considerations:

- Backpressure and flow control: avoid unbounded buffering in memory-constrained C++ processes — use bounded queues and backpressure signals.
- Memory allocation: prefer arena allocators for high-throughput consumers to reduce churn.
- Exactly-once / at-least-once delivery: design idempotent consumers or deduplication to tolerate delivery semantics.



## 4.5 Comparing architectures and selection guidance

How to decide:

- Start by mapping the domain and load profile. If you have a small application and a small team, prefer layered/monolith for simplicity.
- If you need team-level independence, distinct scaling, or language-optimized components (e.g., real-time C++ services), consider microservices.
- For high throughput, real-time streams and decoupling integration points, prefer event-driven architecture with robust partitioning and monitoring.

Example decision: migrating a monolithic analytics app to service-oriented or event-driven architecture

- Evaluate hotspots: where is the system bottleneck (CPU-intensive transforms vs IO-bound web front-ends)?
- Define migration plan: split read-heavy API handling into lightweight HTTP services while moving heavy processing into worker services connected by a message bus.

Operational guidance for C++ systems:

- Standardize packaging and toolchains early to avoid "it works on my machine" (CI builds, reproducible binaries).
- Treat observability/migration as first-class concerns: logs, structured metrics, and tracing are essential.

---

This module focused on how the architectural style affects implementation decisions and trade-offs when C++ is the chosen language. Consider team size, deployment constraints, and the performance profile when selecting an architecture.

Further reading & discussion (optional):

- Map a simple online store to a layered architecture and identify components and boundaries.
- Discuss trade-offs when migrating a monolith to microservices for a CPU-bound C++ data processing application.
