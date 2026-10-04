---
layout: default
clicks: 22
---

<SingleThreadedConcept :step="$clicks" />

<!--
Slide: Why JavaScript is Single-Threaded
Steps 1-5: What is a thread? CPU core analogy
Steps 6-10: The restaurant metaphor (1 chef vs many chefs)
Steps 11-15: DOM race condition — why multi-thread is dangerous
Steps 16-20: The JS solution — single thread + async callbacks
Steps 21-22: V8 engine + libuv diagram intro
-->

---
layout: default
clicks: 22
---

<SingleThreadedDiagram :step="$clicks" />

<!--
Interactive: Thread vs Event Loop
Steps 1-6: Multi-threaded model — threads fighting over DOM
Steps 7-12: Race condition shown live
Steps 13-18: Single-threaded model — one lane, no collision
Steps 19-22: Async callbacks save the day
-->
