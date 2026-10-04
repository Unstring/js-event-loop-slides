# JavaScript Event Loop and Asynchronous Programming

A complete 40-slide masterclass Slidev presentation teaching the JavaScript Concurrency Model, Call Stack, Web APIs, Timers, Microtasks vs Macrotasks, Promises, and Async/Await.

## Presentation Highlights
- **Aspect Ratio**: Tailored for **16:9 widescreen presentation** with optimized vertical layouts, grids, and compact typography to prevent viewport overflow.
- **Incremental Reveal**: Every slide features **10 to 18 progressive `v-click` steps** designed for interactive presenter flow (advancing with spacebar or right arrow).
- **Interactive Vue Components**:
  - `CallStackVisualizer.vue` — Vertical stack showing LIFO push/pop animations.
  - `EventLoopDiagram.vue` — Cyclical architectural visualization connecting Call Stack, Web APIs, Microtasks, Macrotasks, and the Event Loop.
  - `QueueVisualizer.vue` — Dual horizontal queues showing VIP microtask draining and macrotask dequeuing.
  - `CodeTracer.vue` — Execution stepper with code line highlight, live state inspection, and live console terminal.
- **Live In-Browser Code Runners**: Integrated Monaco interactive editor blocks (`{monaco-run}`) on Slides 24 and 39 for live testing during the presentation.
- **Comprehensive Speaker Notes**: Every slide contains speaker notes (`<!-- Speaker Notes ... -->`) viewable in Slidev presenter mode.

---

## 5 Core Learning Outcomes (40 Slides Structure)

1. **Title & Course Agenda** (Slides 1–2)
2. **Outcome 1: Why JavaScript is Single-Threaded** (Slides 3–10)
   - Brendan Eich's 10-day sprint, Netscape Navigator 2.0 (1995)
   - Java Applets vs Lightweight inline scripting
   - The DOM as a shared mutable tree
   - Race conditions on the DOM (thought experiment)
   - Mutexes, deadlocks, and synchronization overhead
   - Head-to-head comparison: JS vs Multi-Threaded Java/C++
   - Single-threaded engine memory (Heap + Stack)
   - The fundamental dilemma: Single-threaded simplicity vs non-blocking performance
3. **Outcome 2: Describe the Call Stack and Event Loop** (Slides 11–18)
   - Call stack mechanics: Stack frames and LIFO
   - Live CallStackVisualizer (4-level nested function trace)
   - Maximum call stack exceeded (infinite recursion & trampolines)
   - Blocking synchronous loops & UI freeze (60 FPS / 16.6ms frame budget)
   - Browser Web APIs & Node.js libuv thread pool
   - Complete Event Loop architectural blueprint
   - Live EventLoopDiagram cycle walkthrough
   - Formal specification of an Event Loop "Tick"
4. **Outcome 3: Understand How setTimeout Works Internally** (Slides 19–26)
   - Debunking the myth: setTimeout is a host API, NOT in ECMAScript
   - 4-stage lifecycle of a timer
   - CodeTracer walkthrough: `console.log('start') ➔ setTimeout(..., 0) ➔ console.log('end')`
   - Zero delay doesn't mean zero wait: 4ms HTML spec clamp & OS clocks
   - Starving timers: Blocked stack delay demonstration
   - Live Monaco code runner with real timer latency measurements
   - setInterval drift vs Recursive setTimeout pattern
   - Time-slicing & chunking heavy calculations to maintain 60 FPS
5. **Outcome 4: Differentiate Between Microtasks and Macrotasks** (Slides 27–33)
   - Two-tier concurrency architecture (VIP queue vs standard queue)
   - The task catalog: Promises, `queueMicrotask`, MutationObserver vs Timers, I/O, UI events
   - The golden rule: Microtasks drain to 100% exhaustion before next macrotask
   - Live QueueVisualizer with interleaved queues
   - The browser render step: Style, layout, and paint timing
   - Predict the output challenge #1 (Promises + Timeouts)
   - Predict the output challenge #2 (Nested microtasks and interleaving)
6. **Outcome 5: Explain Promises and Async/Await** (Slides 34–39)
   - Evolution from Callback Hell to Promises (Inversion of Control)
   - Anatomy of a Promise: States (Pending, Fulfilled, Rejected) and immutability
   - Chaining internals & microtask scheduling
   - Async/Await: Syntactic sugar desugaring to generator/promise coroutines
   - Error handling: `try/catch` with `await` vs `.catch()` & unhandled rejection crashes
   - Live Monaco runner: Promise combinators (`all`, `race`, `allSettled`, `any`)
7. **Wrap-up & Rapid-Fire Quiz** (Slide 40)
   - 5-point outcome architecture recap
   - 4-question rapid-fire interactive quiz with click-to-reveal answers

---

## Installation & Running Locally

### Prerequisites
- Node.js >= 18.0.0
- npm >= 9.0.0

### 1. Install Dependencies
```bash
npm install
```

### 2. Start Local Development Server
```bash
npm run dev
```
Slidev will start the development server (defaulting to `http://localhost:3030`).

### 3. Presenter Shortcuts
- **Space / Right Arrow**: Advance one `v-click` step or slide
- **Left Arrow**: Go back one step
- **`P`**: Toggle Presenter View (with live speaker notes and timer)
- **`O`**: Toggle Slide Overview grid
- **`F`**: Toggle Fullscreen mode
- **`D`**: Toggle Drawing / annotation toolbar

### 4. Build Static Presentation
```bash
npm run build
```
This generates a static single-page application in the `dist/` directory ready for deployment to GitHub Pages, Netlify, or Vercel.

---

## File Structure

```
├── slides.md                    # The 40-slide presentation source
├── style.css                    # Theme customization & 16:9 vertical optimization
├── package.json                 # Dependencies (@slidev/cli, theme, vue)
├── components/
│   ├── CallStackVisualizer.vue  # Interactive LIFO call stack component
│   ├── EventLoopDiagram.vue     # Circular event loop cycle diagram
│   ├── QueueVisualizer.vue      # Dual-queue microtask/macrotask visualizer
│   └── CodeTracer.vue           # Code execution stepper with live console
└── dist/                        # Production build output
```
