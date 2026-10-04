---
layout: default
clicks: 22
---

<CallStackConcept :step="$clicks" />

<!--
Slide: The Call Stack — Concept
Steps 1-6: What is a stack? LIFO analogy (plates)
Steps 7-12: Execution context anatomy (var, scope, this)
Steps 13-18: Step through nested calls code
Steps 19-22: Stack overflow scenario + fix
-->

---
layout: default
clicks: 22
---

<CallStackLive :step="$clicks" />

<!--
Interactive: Call Stack Live Simulator
Steps 1-22: Full step-by-step execution trace of:
  greet('Alice') calling getName() calling getTitle()
  Watch frames push and pop with LIFO highlighting
  Every click = one instruction
-->
