---
layout: default
clicks: 22
---

<QueueConcept :step="$clicks" />

<!--
Chapter 5A: Microtasks vs Macrotasks — Architectural Mechanics
Steps 1-5: The VIP Microtask Queue (Promises, queueMicrotask, MutationObserver)
Steps 6-10: The Macrotask Queue (setTimeout, setInterval, I/O, DOM events)
Steps 11-15: Concurrency Contract (Drain to 0 vs 1 per turn, yield to render)
Steps 16-22: Danger Zone: Microtask Starvation (UI Freezing)
-->

---
layout: default
clicks: 22
---

<QueueLive :step="$clicks" />

<!--
Chapter 5B: Micro vs Macro Live Execution Trace
22 interactive steps stepping through:
2 Timers (0ms) + 2 Chained Promises + Nested Microtask
Watch Microtasks completely bypass and preempt Timers every single time!
-->
