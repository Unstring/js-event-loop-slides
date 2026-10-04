---
layout: default
clicks: 22
---

<PromiseConcept :step="$clicks" />

<!--
Chapter 6A: Promises & async/await Internals
Steps 1-6: The 3 Promise states & immutability guarantee
Steps 7-12: Reaction records & why Promise callbacks are always deferred
Steps 13-17: How async/await desugars into Promise + Generator state machine
Steps 18-22: Heap context suspension & microtask resumption
-->

---
layout: default
clicks: 22
---

<PromiseLive :step="$clicks" />

<!--
Chapter 6B: async / await Live Execution Trace
22 interactive steps stepping through:
Synchronous run -> await null suspension -> context saved to Heap
-> main script continues -> Microtask resumes asyncFn -> Macrotask timer runs last!
-->
