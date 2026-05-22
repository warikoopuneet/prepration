# UPSC Online Exam System Design Interview Notes

## Scenario

Design a scalable online examination platform for a UPSC-style exam where:

* 1 million students start the exam at exactly 9:00 AM
* No waiting room or queue is allowed
* Fairness is more important than raw throughput
* Regional failures must not invalidate the exam
* Users are distributed across India
* Multi-region deployment exists

---

# Key Architectural Principle

This is **not** traditional e-commerce traffic.

This is:

* deterministic
* synchronized
* fairness-sensitive
* politically and legally sensitive

The design must optimize for:

* fairness
* continuity
* durability
* failure isolation

rather than:

* perfect latency
* aggressive cost optimization

---

# Q/A Discussion

---

## Q1. What load balancing approach would you use?

### My Answer

* Use Geo-based routing with Anycast IP and GSLB.
* Route users primarily to nearest region.
* Use weighted balancing according to regional capacity.
* Spill traffic to secondary nearest region only after threshold breach.
* Use ALB inside region with Least Connections algorithm.
* Use cookie/session hashing after session establishment for stickiness.

### Feedback

Strong points:

* Multi-region awareness
* Weighted regional balancing
* Threshold-based spillover
* Session stickiness

Important improvement:

* Explicitly discuss:

  * hysteresis
  * deterministic routing
  * predictive capacity planning
  * anti-oscillation controls

Principal-level phrasing:

> Predictive placement is safer than reactive balancing for synchronized traffic.

---

## Q2. Stateful vs Stateless Sessions

### My Answer

* Keep session state externalized.
* Avoid app-server-local session state.
* If server crashes, another server can resume session.
* Cookie can contain region metadata.
* New server can fetch session from old region.

### Feedback

Strong points:

* Correct use of externalized state
* Resilience-oriented thinking
* Session continuity focus

Principal-level improvements:

* Mention:

  * append-only event logs
  * idempotent answer APIs
  * monotonic versioning
  * replicated durable append before ACK

Important refinement:
Cross-region fetch should be exceptional, not default.

---

## Q3. Preventing Regional Overload Oscillation

### My Answer

* Preallocate capacity using expected regional candidate distribution.
* Use historical load prediction.
* Spillover only after threshold breach.
* Keep lower thresholds when traffic distribution is unknown.
* Use autoscaling as secondary support.

### Feedback

Strong points:

* Recognized deterministic workload
* Predictive provisioning
* Capacity-aware routing
* Threshold-based balancing

Principal-level improvements:

* Explicitly mention:

  * hysteresis
  * failover headroom
  * anti-flapping logic
  * regional reserve capacity

Important principle:

> Autoscaling is not the primary defense for a synchronized 9 AM spike.

---

## Q4. First 5 Minutes of Traffic

### My Answer

#### DNS & CDN

* DNS TTL = 5 mins
* Prewarm ISP DNS caches before exam
* Warm CDN before exam start

#### Authentication

* Aggressively scale auth service
* Preload auth/session data into memory

#### Question Paper Delivery

* Avoid pre-caching paper too early
* Use CDN request coalescing
* Prevent question paper leakage

#### Answer Submission

* Use event-sourcing approach
* Accept answer before DB persistence
* Async persistence downstream

#### WebSockets

* Use WebSockets for continuity and fairness
* Pause exam during disconnect

### Feedback

Strong points:

* CDN and cache prewarming
* Security-aware paper delivery
* Event-driven persistence
* Fairness-oriented reconnect logic

Principal-level improvements:

* Mention:

  * TLS warmup
  * replicated durable append
  * reconnect storms
  * local client checkpointing
  * heartbeat optimization

Important principle:

> Recovery traffic is often more dangerous than steady-state traffic.

---

## Q5. Mumbai Region Failure at 9:07 AM

### My Answer

* Pause timer during disconnect.
* Reconnect through GSLB.
* Redirect users to next healthy region.
* Sync session state to new region.
* Avoid rebalancing users back after recovery.

### Feedback

Strong points:

* Fairness-first design
* Sticky failover
* Avoiding rebalancing storms
* Session continuity awareness

Principal-level improvements:

* Mention:

  * exponential reconnect backoff
  * reconnect jitter
  * graceful degradation
  * reserved failover capacity
  * client-side encrypted checkpoints

Critical insight:

> Failover traffic can kill healthy regions if reconnects are uncontrolled.

---

## Q6. Metrics and Monitoring

### My Answer

### Infra Metrics

* New connections/sec
* Active connections
* Event bus lag
* Server load
* Cache/state cluster health

### User/Fairness Metrics

* Answer submissions
* Submissions per IP
* Multiple device usage
* Reconnect attempts
* Login failures
* L7 rejection metrics

### Startup Metrics

* Active connections/server
* Connection attempts
* Connection retries
* Server response times

### Feedback

Strong points:

* Operational thinking
* Distinguishing startup vs steady-state telemetry
* Event bus lag awareness
* Security/fairness telemetry

Principal-level improvements:

* Explicit fairness metrics:

  * timer skew
  * reconnect recovery duration
  * answer ACK latency
  * paused-session duration
  * p95/p99 latency
  * region-wise fairness drift

Important principle:

> Fairness metrics matter more than raw infrastructure metrics.

---

# Overall Interview Rating

## Overall Rating: 7.5 to 8 / 10 for Principal Engineer

### Interpretation

* Strong Staff / Senior Staff signal
* Borderline-to-strong Principal signal
* Strong operational and distributed systems intuition
* Good failure-awareness
* Good fairness-oriented reasoning
* Strong practical instincts

---

# Biggest Strengths

* Failure-oriented thinking
* Externalized session architecture
* Operational awareness
* Understanding synchronized traffic
* Event-driven persistence understanding
* Fairness-aware design
* Distinguishing predictable vs unpredictable traffic

---

# Biggest Gaps to Reach Strong Principal Level

## 1. State Explicit Invariants Early

Example:

* No acknowledged answer may be lost
* No user loses exam time due to infra
* Regional failure cannot impact >X% users
* Timer skew must remain <Y seconds

---

## 2. Speak More Quantitatively

Add:

* throughput assumptions
* retry amplification estimates
* capacity math
* heartbeat load calculations
* failover percentage calculations

Example:

> 1M WebSockets with reconnect storms can multiply ingress traffic 5-10x.

---

## 3. Discuss Intentional Degradation

Principal candidates define:

* what gets sacrificed first
* what remains protected

Example:

* disable analytics first
* reduce heartbeat frequency
* prioritize answer submission over telemetry

---

## 4. Use More Precise Distributed Systems Terminology

Examples:

* hysteresis
* quorum durability
* tail latency
* backpressure
* blast radius
* at-least-once semantics
* idempotency
* failover headroom

---

# Principal-Level Interview Framing Strategy

Start future interviews with:

## Step 1: Define Invariants

What absolutely must remain true?

## Step 2: Define Traffic Shape

Deterministic or stochastic?

## Step 3: Define Failure Model

What happens when regions partially fail?

## Step 4: Define Graceful Degradation

What gets sacrificed first?

## Step 5: Define Fairness Metrics

How do we detect unfairness before outage?

This framing makes the discussion feel principal-level immediately.

---

# Final Takeaway

The strongest signal demonstrated during this interview:

> Thinking in terms of continuity, fairness, recovery, and operational behavior instead of only technologies.

That is difficult to fake and is usually developed through real-world production exposure.

The next level is becoming more explicit about:

* invariants
* quantitative reasoning
* tradeoffs
* failure envelopes
* graceful degradation policies

That transition is what typically separates:

* strong Staff Engineers
  from
* strong Principal Engineers.
