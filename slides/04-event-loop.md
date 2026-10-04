---
layout: default
clicks: 22
---

<EventLoopConcept :step="$clicks" />

<!--
Chapter 3A: Event Loop Concept
Steps 1-6: What is the event loop? The infinite heartbeat
Steps 7-12: 4 phases: Stack check → Microtasks → Render → Macrotask
Steps 13-18: Host environments: Browser (Web APIs) vs Node.js (libuv)
Steps 19-22: The full lifecycle question + answer
-->

---
layout: default
clicks: 22
---

<EventLoopLive :step="$clicks" />

<!--
Chapter 3B: Event Loop — Full Lifecycle Interactive
22 click steps tracing complete lifecycle:
console.log → setTimeout → Promise → queueMicrotask → console.log
→ Stack empty → drain microtasks → render check → macrotask
Every click highlights the active zone
-->
