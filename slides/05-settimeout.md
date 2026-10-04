---
layout: default
clicks: 22
---

<SetTimeoutConcept :step="$clicks" />

<!--
Chapter 4A: setTimeout internals
Steps 1-6: What setTimeout ACTUALLY does (not sleep!)
Steps 7-12: The Min-Heap timer queue inside libuv/browser
Steps 13-18: The 0ms myth — minimum NOT actual delay
Steps 19-22: 4ms HTML5 clamping rule + background tab throttle
-->

---
layout: default
clicks: 22
---

<SetTimeoutLive :step="$clicks" />

<!--
Chapter 4B: setTimeout Live Simulator
22 steps showing:
- Code executes synchronously while timer ticks in background
- What happens when stack is busy for 2000ms
- Timer fires into macrotask queue, waits for stack
- Actual delay vs specified delay gap
-->
