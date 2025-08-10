## 1.1 Reliability
**Goal:** System works correctly even when things go wrong.
**Key mindset:** Failures are inevitable — design so they don’t become disasters.

Reliability isn’t about zero faults, it’s about preventing faults from becoming failures.

*   **Fault ≠ Failure**
*   **Fault** → one part of the system misbehaves (e.g., disk crash).
*   **Failure** → the whole system stops delivering its service to the user.

Don’t try to eliminate all faults — contain them.

Systems that handle faults without failing are called fault-tolerant or resilient.

### Types of Faults

#### Hardware Faults
*   **Examples:** Hard disks crash, power failures, cables unplug.
*   **Scale example:** 10,000 disks → expect 1 to fail daily.
*   **Old-school fix:** RAID, dual power supplies, backup generators.
*   **Modern need:** Software-level fault tolerance (replication, failover).
*   **Example:** 10 servers running → one fails, others keep going. Rolling upgrades without full downtime.

#### Software Errors
Harder than hardware faults — they can hit all machines at once.
*   **Examples:**
    *   Bug triggered by rare input (e.g., 2012 Leap Second bug).
    *   Memory leak causing OOM crashes.
*   **Prevention:**
    *   Strong test coverage (unit, integration, load tests).
    *   Isolate processes so one crash ≠ whole system crash.
    *   Allow auto-restarts (process supervision).
    *   Monitor + track metrics to spot problems early.

#### Human Errors
Biggest cause of outages.
*   **Examples:**
    *   Wrong config pushed to prod.
    *   Accidental data deletion.
*   **Prevention:**
    *   Separate prod from staging.
    *   Read-only prod creds when possible.
    *   Clear, safe admin interfaces.
    *   Easy rollback + recovery tools.
    *   Real-time monitoring to catch bad deploys quickly.

### Why Reliability Matters:
Downtime = lost money, reputation damage, legal risks.

Even a “toy” app can cause irreversible harm if it loses user data.

Sometimes we trade reliability for speed/cost (e.g., prototypes) — but be aware of the risk.

## 1.2 Scalability
**Goal:** System should work reliably on increasing load.
### How to Think About Load
Every system has load parameters — things that grow when usage grows:
*   Requests per second (RPS)
*   Number of active users at the same time
*   Size of stored data
*   Number of events/messages per second

Scaling isn’t about a magic “scale” button. It’s about finding which part of your system is hitting limits and fixing that.

### Twitter Example
**Problem:** Two main operations:
*   Posting a tweet (write-heavy)
*   Viewing the timeline (read-heavy)

#### Scaling Approaches:

##### Read-time fan-out
*   Store all tweets in one place (like a giant DB table).
*   When a user opens their timeline → fetch all tweets from people they follow at read time.
*   **Pros:** Writing a tweet is super fast — just one insert.
*   **Cons:** Reading is slow — if a user follows 2k people, you fetch + merge all their tweets at once.

##### Write-time fan-out
*   When someone tweets, copy it into each follower’s personal timeline store at write time.
*   **Pros:** Reading is instant — just read your own stored list.
*   **Cons:** If a celebrity with 20M followers tweets, the system must write 20M copies at once.

##### Hybrid approach
*   Normal users = write-time fan-out (fast reads).
*   Celebrities = read-time fan-out (avoid massive writes).
This hybrid design is common in real-world systems: one method for most users, another for extreme cases.

### Performance Metrics
*   **Throughput:** How many requests or MB your system handles per second.
*   **Latency:** How long a request takes from send → response.

### Why Percentiles Matter
Average latency hides slow requests.
*   **Example:**
    *   p95 = 95% of requests are faster than this number.
    *   p99 = only 1% are slower.
Users notice slow outliers more than fast averages.

### Tail Latency Amplification
In distributed systems, a single user request often triggers multiple backend calls (to databases, APIs, caches, etc.).
The overall request time is as slow as the slowest backend call.
Even if 9/10 backend calls are fast, the one slow call drags down the entire response.
*   **Example:** Loading a Twitter feed calls:
    *   Tweet storage service
    *   Profile service
    *   Image service
    If the image service takes 2 seconds, the whole page feels slow — even if tweets and profiles were instant.
This effect gets worse as the number of backend calls increases → more chances that at least one is slow.
**Common mitigations:** request hedging, caching, reducing fan-out, async loading of non-critical parts.

### Parallelism
CPU speeds (GHz) are not getting much faster anymore.
Instead, processors now have more cores — meaning they can do many things at the same time.
To get faster, you must use those extra cores:
*   Break big jobs into smaller pieces and run them in parallel.
    *   **Example:** Processing 1M log lines → split into 10 tasks of 100k lines each, run on different cores.
*   Use event-driven systems or async processing so one slow task doesn’t block others.
    *   **Example:** Kafka consumer groups — each consumer handles different partitions at the same time.

## 1.3 Maintainability
**Goal:** Keep the system easy to operate, understand, and change for many years, even as people, tools, and requirements change.
### Why it matters:
*   Most cost is in maintaining, not building.
*   Hard-to-maintain systems slow down dev speed, increase bugs, and make outages harder to fix.

### Three Sub-Pillars

#### 1. Operability — Make it painless to run the system.
*   **Monitoring + Alerting** → Detect issues before users report them.
*   **Automation** → CI/CD pipelines, auto-scaling, auto-healing.
*   **Clear Metrics & Logs** → Traceable and consistent formats (e.g., structured logging, OpenTelemetry).
*   **Safe Upgrades** → Rolling deploys, feature flags, backward compatibility for APIs.
*   **Runbooks** → Document common failures and recovery steps.

#### 2. Simplicity — Fight accidental complexity.
*   Fewer moving parts → fewer places for bugs.
*   Use clean abstractions to hide details that don’t matter to the caller.
    *   **Example:** SQL hides storage engine internals; REST API hides internal logic.
*   Refactor messy code early — complexity compounds like debt.

#### 3. Evolvability — Make change cheap and safe.
*   Expect requirements, laws, and tech to change.
*   Use modular designs → change one part without breaking the rest.
*   Use tests as safety nets → TDD, integration tests, and regression suites.
*   Favor backward compatibility when evolving APIs or DB schemas.
*   Keep dependencies up to date to avoid upgrade “big bang” later.
