<script setup lang="ts">
import { computed } from 'vue'
const props = defineProps<{ step?: number }>()
const s = computed(() => Math.min(Math.max(props.step ?? 0, 0), 22))

// The full program being traced:
// console.log('1. Start')
// setTimeout(() => console.log('4. Timeout'), 10)
// Promise.resolve().then(() => console.log('3. Promise'))
// queueMicrotask(() => console.log('3b. qMT'))
// console.log('2. End')

interface ELState {
  stack: string[]
  webapi: string[]
  microtasks: string[]
  macrotasks: string[]
  output: string[]
  renderActive: boolean
  activeZone: 'stack' | 'webapi' | 'micro' | 'macro' | 'render' | 'idle'
  note: string
  angle: number
}

const states: ELState[] = [
  // 0 initial
  { stack: [], webapi: [], microtasks: [], macrotasks: [], output: [], renderActive: false, activeZone: 'idle', note: '▶ Program starts. All queues empty.', angle: 0 },
  // 1 global starts
  { stack: ['main()'], webapi: [], microtasks: [], macrotasks: [], output: [], renderActive: false, activeZone: 'stack', note: 'Global script → main() pushed to Call Stack.', angle: 10 },
  // 2 log start
  { stack: ['main()', "log('1. Start')"], webapi: [], microtasks: [], macrotasks: [], output: [], renderActive: false, activeZone: 'stack', note: "console.log('1. Start') pushed — runs synchronously.", angle: 20 },
  // 3 log start pops, output
  { stack: ['main()'], webapi: [], microtasks: [], macrotasks: [], output: ['1. Start'], renderActive: false, activeZone: 'stack', note: "✓ '1. Start' printed. console.log popped.", angle: 40 },
  // 4 setTimeout
  { stack: ['main()', 'setTimeout(cb, 10)'], webapi: [], microtasks: [], macrotasks: [], output: ['1. Start'], renderActive: false, activeZone: 'stack', note: 'setTimeout(cb, 10) called. V8 hands timer to Web APIs.', angle: 55 },
  // 5 setTimeout hands off
  { stack: ['main()'], webapi: ['⏱ Timer(cb) 10ms'], microtasks: [], macrotasks: [], output: ['1. Start'], renderActive: false, activeZone: 'webapi', note: 'Timer handed to Browser C++ thread. Stack does NOT wait!', angle: 80 },
  // 6 Promise.resolve().then
  { stack: ['main()', 'Promise.resolve().then(p1)'], webapi: ['⏱ Timer(cb) 8ms'], microtasks: [], macrotasks: [], output: ['1. Start'], renderActive: false, activeZone: 'stack', note: 'Promise.resolve() → already settled → schedules p1 as microtask.', angle: 100 },
  // 7 p1 enters microtask queue
  { stack: ['main()'], webapi: ['⏱ Timer(cb) 6ms'], microtasks: ['p1: log(Promise)'], macrotasks: [], output: ['1. Start'], renderActive: false, activeZone: 'micro', note: '⭐ p1 enters Microtask VIP Queue. Promise callbacks always go here!', angle: 120 },
  // 8 queueMicrotask
  { stack: ['main()', 'queueMicrotask(m2)'], webapi: ['⏱ Timer(cb) 4ms'], microtasks: ['p1: log(Promise)', 'm2: log(qMT)'], macrotasks: [], output: ['1. Start'], renderActive: false, activeZone: 'micro', note: 'queueMicrotask(m2) → m2 appended to microtask queue.', angle: 140 },
  // 9 log end
  { stack: ['main()', "log('2. End')"], webapi: ['⏱ Timer(cb) 2ms'], microtasks: ['p1: log(Promise)', 'm2: log(qMT)'], macrotasks: [], output: ['1. Start'], renderActive: false, activeZone: 'stack', note: "console.log('2. End') pushed. Last synchronous call.", angle: 155 },
  // 10 log end outputs
  { stack: ['main()'], webapi: ['⏱ Timer(cb) 1ms'], microtasks: ['p1: log(Promise)', 'm2: log(qMT)'], macrotasks: [], output: ['1. Start', '2. End'], renderActive: false, activeZone: 'stack', note: "'2. End' printed. Synchronous code almost done.", angle: 170 },
  // 11 main pops — CRITICAL
  { stack: [], webapi: ['⏱ Timer(cb) 0ms → FIRED'], microtasks: ['p1: log(Promise)', 'm2: log(qMT)'], macrotasks: ['cb: log(Timeout)'], output: ['1. Start', '2. End'], renderActive: false, activeZone: 'idle', note: '🔑 Stack is EMPTY! Timer fired → cb enters Macrotask queue. Event Loop wakes!', angle: 180 },
  // 12 EL checks microtasks
  { stack: [], webapi: [], microtasks: ['p1: log(Promise)', 'm2: log(qMT)'], macrotasks: ['cb: log(Timeout)'], output: ['1. Start', '2. End'], renderActive: false, activeZone: 'micro', note: '⭐ PHASE 1: Event Loop drains ALL microtasks before anything else!', angle: 200 },
  // 13 run p1
  { stack: ["p1: log('Promise')"], webapi: [], microtasks: ['m2: log(qMT)'], macrotasks: ['cb: log(Timeout)'], output: ['1. Start', '2. End'], renderActive: false, activeZone: 'micro', note: 'Dequeue p1 → pushed to stack → executes console.log(Promise).', angle: 215 },
  // 14 p1 output
  { stack: [], webapi: [], microtasks: ['m2: log(qMT)'], macrotasks: ['cb: log(Timeout)'], output: ['1. Start', '2. End', '3. Promise'], renderActive: false, activeZone: 'micro', note: "'3. Promise' logged. p1 frame popped. More microtasks remain!", angle: 230 },
  // 15 run m2
  { stack: ["m2: log('qMT')"], webapi: [], microtasks: [], macrotasks: ['cb: log(Timeout)'], output: ['1. Start', '2. End', '3. Promise'], renderActive: false, activeZone: 'micro', note: 'Dequeue m2 → executes. Microtask queue draining...', angle: 245 },
  // 16 m2 output
  { stack: [], webapi: [], microtasks: [], macrotasks: ['cb: log(Timeout)'], output: ['1. Start', '2. End', '3. Promise', '3b. qMT'], renderActive: false, activeZone: 'micro', note: "'3b. qMT' logged. Microtask queue is 100% DRAINED!", angle: 260 },
  // 17 render check
  { stack: [], webapi: [], microtasks: [], macrotasks: ['cb: log(Timeout)'], output: ['1. Start', '2. End', '3. Promise', '3b. qMT'], renderActive: true, activeZone: 'render', note: '🎨 PHASE 2: Render opportunity check. V-Sync pulse? → rAF → Style → Layout → Paint.', angle: 290 },
  // 18 render done
  { stack: [], webapi: [], microtasks: [], macrotasks: ['cb: log(Timeout)'], output: ['1. Start', '2. End', '3. Promise', '3b. qMT'], renderActive: false, activeZone: 'macro', note: 'Render complete. PHASE 3: Event Loop picks ONE macrotask.', angle: 310 },
  // 19 macrotask runs
  { stack: ["cb: log('Timeout')"], webapi: [], microtasks: [], macrotasks: [], output: ['1. Start', '2. End', '3. Promise', '3b. qMT'], renderActive: false, activeZone: 'macro', note: 'Dequeue cb → pushed to stack. setTimeout callback FINALLY runs!', angle: 330 },
  // 20 timeout output
  { stack: [], webapi: [], microtasks: [], macrotasks: [], output: ['1. Start', '2. End', '3. Promise', '3b. qMT', '4. Timeout'], renderActive: false, activeZone: 'idle', note: "'4. Timeout' logged. cb popped. All queues empty.", angle: 350 },
  // 21 summary
  { stack: [], webapi: [], microtasks: [], macrotasks: [], output: ['1. Start', '2. End', '3. Promise', '3b. qMT', '4. Timeout'], renderActive: false, activeZone: 'idle', note: '✅ Complete cycle! Order: Sync → Microtasks → Render → Macrotask.', angle: 360 },
  // 22 quiz
  { stack: [], webapi: [], microtasks: [], macrotasks: [], output: ['1. Start', '2. End', '3. Promise', '3b. qMT', '4. Timeout'], renderActive: false, activeZone: 'idle', note: '🏆 This order is THE LAW of JS async execution. Everything else follows from it.', angle: 360 },
]

const cur = computed(() => states[Math.min(s.value, states.length - 1)])

const zoneClass = (zone: string) => {
  const active = cur.value.activeZone === zone
  const map: Record<string, string> = {
    stack: 'bg-sky-50 border-sky-500 ring-2 ring-sky-300 shadow-md scale-[1.02]',
    webapi: 'bg-emerald-50 border-emerald-500 ring-2 ring-emerald-300 shadow-md scale-[1.02]',
    micro: 'bg-violet-50 border-violet-500 ring-2 ring-violet-300 shadow-md scale-[1.02]',
    render: 'bg-rose-50 border-rose-500 ring-2 ring-rose-300 shadow-md scale-[1.02]',
    macro: 'bg-amber-50 border-amber-500 ring-2 ring-amber-300 shadow-md scale-[1.02]',
    idle: '',
  }
  return active ? map[zone] : 'bg-white border-slate-200 opacity-80'
}
</script>

<template>
  <div class="h-full flex flex-col gap-1.5 select-none">
    <div class="flex items-center gap-3 pb-1 border-b-2 border-slate-200">
      <span class="text-2xl font-black" :class="cur.activeZone !== 'idle' ? 'animate-spin' : ''">↻</span>
      <div>
        <h2 class="text-xl font-black text-slate-900 leading-tight">Event Loop — Full Lifecycle</h2>
        <p class="text-xs text-slate-500">Chapter 3 of 5 · Interactive</p>
      </div>
      <div class="ml-auto px-2 py-1 rounded-lg bg-sky-600 text-white text-xs font-bold">Step {{ s }}/22</div>
    </div>

    <!-- Action banner -->
    <div
      class="rounded-xl px-3 py-1.5 text-xs font-bold border-2 flex items-center gap-2 transition-all duration-300"
      :class="{
        'bg-sky-50 border-sky-400 text-sky-900': cur.activeZone === 'stack',
        'bg-emerald-50 border-emerald-400 text-emerald-900': cur.activeZone === 'webapi',
        'bg-violet-50 border-violet-400 text-violet-900': cur.activeZone === 'micro',
        'bg-rose-50 border-rose-400 text-rose-900': cur.activeZone === 'render',
        'bg-amber-50 border-amber-400 text-amber-900': cur.activeZone === 'macro',
        'bg-slate-50 border-slate-200 text-slate-700': cur.activeZone === 'idle',
      }"
    >
      <span class="animate-pulse">▶</span>
      {{ cur.note }}
    </div>

    <div class="grid grid-cols-12 gap-2 flex-1 min-h-0">
      <!-- Call Stack (3 cols) -->
      <div class="col-span-3 border-2 rounded-xl p-2 flex flex-col gap-1 transition-all duration-300" :class="zoneClass('stack')">
        <div class="text-[10px] font-black text-slate-700 flex items-center justify-between">
          <span>⚡ Call Stack</span>
          <span class="bg-sky-100 text-sky-800 px-1 rounded text-[9px]">{{ cur.stack.length }} frames</span>
        </div>
        <div class="flex-1 flex flex-col-reverse gap-1">
          <div v-if="cur.stack.length === 0" class="flex items-center justify-center h-full">
            <span class="text-[10px] text-slate-400 italic">empty</span>
          </div>
          <div
            v-for="(f, i) in cur.stack" :key="f + i"
            class="rounded px-1.5 py-0.5 text-[10px] font-bold border flex justify-between"
            :class="i === cur.stack.length - 1 ? 'bg-sky-600 text-white border-sky-700 animate-pulse' : 'bg-sky-50 border-sky-200 text-sky-900'"
          >
            <span class="truncate">{{ f }}</span>
            <span class="text-[8px] ml-1 shrink-0">{{ i === cur.stack.length - 1 ? 'TOP' : '' }}</span>
          </div>
        </div>
      </div>

      <!-- Center: Event Loop wheel + Web APIs (4 cols) -->
      <div class="col-span-4 flex flex-col gap-2">
        <!-- Web APIs -->
        <div class="border-2 rounded-xl p-2 flex-1 flex flex-col gap-1 transition-all duration-300" :class="zoneClass('webapi')">
          <div class="text-[10px] font-black text-slate-700">🌐 Web APIs / libuv (OS Threads)</div>
          <div class="flex-1">
            <div v-if="cur.webapi.length === 0" class="text-[10px] text-slate-400 italic">threads idle</div>
            <div
              v-for="w in cur.webapi" :key="w"
              class="text-[10px] font-bold rounded px-1.5 py-0.5 bg-emerald-100 border border-emerald-400 text-emerald-900 mb-0.5"
            >{{ w }}</div>
          </div>
        </div>

        <!-- Event Loop Hub -->
        <div class="flex items-center justify-center py-1">
          <div
            class="w-16 h-16 rounded-full border-4 flex flex-col items-center justify-center transition-all duration-300 shadow-lg"
            :class="cur.activeZone !== 'idle' ? 'border-amber-500 bg-amber-100 ring-4 ring-amber-300' : 'border-slate-300 bg-slate-100'"
          >
            <div
              class="text-xl font-black text-amber-600 transition-all duration-500"
              :style="{ transform: `rotate(${cur.angle}deg)` }"
            >↻</div>
            <div class="text-[8px] font-black text-slate-800">EVENT<br/>LOOP</div>
          </div>
        </div>
      </div>

      <!-- Right: Queues + Output (5 cols) -->
      <div class="col-span-5 flex flex-col gap-1.5">
        <!-- Microtask Queue -->
        <div class="border-2 rounded-xl p-2 flex-1 flex flex-col gap-1 transition-all duration-300" :class="zoneClass('micro')">
          <div class="text-[10px] font-black text-slate-700 flex items-center justify-between">
            <span>⭐ Microtasks (VIP)</span>
            <span class="bg-violet-100 text-violet-800 px-1 rounded text-[9px]">100% drain first</span>
          </div>
          <div class="flex-1">
            <div v-if="cur.microtasks.length === 0" class="text-[10px] text-slate-400 italic">queue drained ✓</div>
            <div
              v-for="m in cur.microtasks" :key="m"
              class="text-[10px] font-bold rounded px-1.5 py-0.5 bg-violet-600 text-white mb-0.5 animate-pulse"
            >👑 {{ m }}</div>
          </div>
        </div>

        <!-- Render -->
        <div class="border-2 rounded-xl p-2 transition-all duration-300" :class="zoneClass('render')">
          <div class="text-[10px] font-black text-slate-700 flex items-center justify-between">
            <span>🎨 Render (16.6ms)</span>
            <span :class="cur.renderActive ? 'text-rose-600 font-black animate-pulse' : 'text-slate-400'">{{ cur.renderActive ? 'ACTIVE' : 'idle' }}</span>
          </div>
          <div class="text-[9px] text-slate-500 mt-0.5">rAF → Style → Layout → Paint → Composite</div>
        </div>

        <!-- Macrotask Queue -->
        <div class="border-2 rounded-xl p-2 flex-1 flex flex-col gap-1 transition-all duration-300" :class="zoneClass('macro')">
          <div class="text-[10px] font-black text-slate-700 flex items-center justify-between">
            <span>⏳ Macrotasks</span>
            <span class="bg-amber-100 text-amber-800 px-1 rounded text-[9px]">1 per turn</span>
          </div>
          <div class="flex-1">
            <div v-if="cur.macrotasks.length === 0" class="text-[10px] text-slate-400 italic">queue empty</div>
            <div
              v-for="t in cur.macrotasks" :key="t"
              class="text-[10px] font-bold rounded px-1.5 py-0.5 bg-amber-500 text-amber-950 mb-0.5"
            >⏱ {{ t }}</div>
          </div>
        </div>

        <!-- Console Output -->
        <div class="border-2 border-slate-300 rounded-xl p-2 bg-slate-900">
          <div class="text-[10px] font-black text-slate-400 mb-1">📟 Console</div>
          <div v-if="cur.output.length === 0" class="text-[10px] text-slate-600 italic">awaiting...</div>
          <div
            v-for="(o, i) in cur.output" :key="i"
            class="font-mono text-emerald-400 font-bold text-[10px]"
          >▶ "{{ o }}"</div>
        </div>
      </div>
    </div>
  </div>
</template>
