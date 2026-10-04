<script setup lang="ts">
import { computed } from 'vue'

const props = defineProps<{ step?: number }>()
const s = computed(() => Math.min(Math.max(props.step ?? 0, 0), 22))

interface ELState {
  codeLine: number
  stack: string[]
  webapi: string[]
  microtasks: string[]
  macrotasks: string[]
  output: string[]
  renderActive: boolean
  activeZone: 'code' | 'stack' | 'webapi' | 'micro' | 'macro' | 'render' | 'idle'
  note: string
  angle: number
}

const states: ELState[] = [
  // 0 initial
  { codeLine: 0, stack: [], webapi: [], microtasks: [], macrotasks: [], output: [], renderActive: false, activeZone: 'idle', note: '▶ Program loaded. Ready to trace the complete Event Loop lifecycle.', angle: 0 },
  // 1 global starts
  { codeLine: 1, stack: ['main()'], webapi: [], microtasks: [], macrotasks: [], output: [], renderActive: false, activeZone: 'stack', note: 'Global script enters Call Stack as main().', angle: 15 },
  // 2 log start
  { codeLine: 1, stack: ['main()', "log('1. Start')"], webapi: [], microtasks: [], macrotasks: [], output: [], renderActive: false, activeZone: 'stack', note: "Line 1: console.log('1. Start') executes synchronously.", angle: 30 },
  // 3 log start pops, output
  { codeLine: 1, stack: ['main()'], webapi: [], microtasks: [], macrotasks: [], output: ['1. Start'], renderActive: false, activeZone: 'stack', note: "✓ '1. Start' logged. Stack frame popped.", angle: 45 },
  // 4 setTimeout
  { codeLine: 2, stack: ['main()', 'setTimeout(cb, 10)'], webapi: [], microtasks: [], macrotasks: [], output: ['1. Start'], renderActive: false, activeZone: 'stack', note: 'Line 2: setTimeout(cb, 10ms) hands timer off to Web APIs.', angle: 60 },
  // 5 setTimeout hands off
  { codeLine: 2, stack: ['main()'], webapi: ['⏱ Timer(cb) 10ms'], microtasks: [], macrotasks: [], output: ['1. Start'], renderActive: false, activeZone: 'webapi', note: 'Timer runs on Host C++ OS thread. Main thread does NOT wait!', angle: 75 },
  // 6 Promise.resolve().then
  { codeLine: 3, stack: ['main()', 'Promise.resolve().then(p1)'], webapi: ['⏱ Timer(cb) 8ms'], microtasks: [], macrotasks: [], output: ['1. Start'], renderActive: false, activeZone: 'stack', note: 'Line 3: Promise fulfills immediately; registers p1 callback.', angle: 90 },
  // 7 p1 enters microtask queue
  { codeLine: 3, stack: ['main()'], webapi: ['⏱ Timer(cb) 6ms'], microtasks: ['p1: log(Promise)'], macrotasks: [], output: ['1. Start'], renderActive: false, activeZone: 'micro', note: '⭐ p1 enters VIP Microtask Queue! Promise reactions go here.', angle: 110 },
  // 8 queueMicrotask
  { codeLine: 4, stack: ['main()', 'queueMicrotask(m2)'], webapi: ['⏱ Timer(cb) 4ms'], microtasks: ['p1: log(Promise)', 'm2: log(qMT)'], macrotasks: [], output: ['1. Start'], renderActive: false, activeZone: 'micro', note: 'Line 4: queueMicrotask(m2) adds m2 to the VIP Microtask Queue.', angle: 130 },
  // 9 log end
  { codeLine: 5, stack: ['main()', "log('2. End')"], webapi: ['⏱ Timer(cb) 2ms'], microtasks: ['p1: log(Promise)', 'm2: log(qMT)'], macrotasks: [], output: ['1. Start'], renderActive: false, activeZone: 'stack', note: "Line 5: console.log('2. End') executes synchronously.", angle: 150 },
  // 10 log end outputs
  { codeLine: 5, stack: ['main()'], webapi: ['⏱ Timer(cb) 1ms'], microtasks: ['p1: log(Promise)', 'm2: log(qMT)'], macrotasks: [], output: ['1. Start', '2. End'], renderActive: false, activeZone: 'stack', note: "✓ '2. End' logged. End of synchronous statements.", angle: 165 },
  // 11 main pops — CRITICAL
  { codeLine: 0, stack: [], webapi: ['⏱ Timer(cb) fired!'], microtasks: ['p1: log(Promise)', 'm2: log(qMT)'], macrotasks: ['cb: log(Timeout)'], output: ['1. Start', '2. End'], renderActive: false, activeZone: 'idle', note: '🔑 Call Stack is EMPTY! Timer fired → cb entered Macrotask Queue.', angle: 180 },
  // 12 EL checks microtasks
  { codeLine: 0, stack: [], webapi: [], microtasks: ['p1: log(Promise)', 'm2: log(qMT)'], macrotasks: ['cb: log(Timeout)'], output: ['1. Start', '2. End'], renderActive: false, activeZone: 'micro', note: '⭐ PHASE 1: Event Loop drains ALL Microtasks before anything else!', angle: 200 },
  // 13 run p1
  { codeLine: 3, stack: ["p1: log('Promise')"], webapi: [], microtasks: ['m2: log(qMT)'], macrotasks: ['cb: log(Timeout)'], output: ['1. Start', '2. End'], renderActive: false, activeZone: 'stack', note: 'Dequeue p1 → pushes to Call Stack → runs console.log.', angle: 220 },
  // 14 p1 output
  { codeLine: 3, stack: [], webapi: [], microtasks: ['m2: log(qMT)'], macrotasks: ['cb: log(Timeout)'], output: ['1. Start', '2. End', '3. Promise'], renderActive: false, activeZone: 'micro', note: "✓ '3. Promise' printed. Stack popped. Microtasks still remain!", angle: 240 },
  // 15 run m2
  { codeLine: 4, stack: ["m2: log('qMT')"], webapi: [], microtasks: [], macrotasks: ['cb: log(Timeout)'], output: ['1. Start', '2. End', '3. Promise'], renderActive: false, activeZone: 'stack', note: 'Dequeue m2 → pushed to Call Stack → executes.', angle: 260 },
  // 16 m2 output
  { codeLine: 4, stack: [], webapi: [], microtasks: [], macrotasks: ['cb: log(Timeout)'], output: ['1. Start', '2. End', '3. Promise', '3b. qMT'], renderActive: false, activeZone: 'micro', note: "✓ '3b. qMT' printed. VIP Microtask queue is now 100% DRAINED!", angle: 280 },
  // 17 render check
  { codeLine: 0, stack: [], webapi: [], microtasks: [], macrotasks: ['cb: log(Timeout)'], output: ['1. Start', '2. End', '3. Promise', '3b. qMT'], renderActive: true, activeZone: 'render', note: '🎨 PHASE 2: Render Opportunity! V-Sync pulse → Style, Layout, Paint.', angle: 300 },
  // 18 render done
  { codeLine: 0, stack: [], webapi: [], microtasks: [], macrotasks: ['cb: log(Timeout)'], output: ['1. Start', '2. End', '3. Promise', '3b. qMT'], renderActive: false, activeZone: 'macro', note: 'Render complete. PHASE 3: Event Loop picks exactly ONE Macrotask.', angle: 320 },
  // 19 macrotask runs
  { codeLine: 2, stack: ["cb: log('Timeout')"], webapi: [], microtasks: [], macrotasks: [], output: ['1. Start', '2. End', '3. Promise', '3b. qMT'], renderActive: false, activeZone: 'stack', note: 'Dequeue cb → pushed to Call Stack. setTimeout callback runs!', angle: 340 },
  // 20 timeout output
  { codeLine: 2, stack: [], webapi: [], microtasks: [], macrotasks: [], output: ['1. Start', '2. End', '3. Promise', '3b. qMT', '4. Timeout'], renderActive: false, activeZone: 'idle', note: "✓ '4. Timeout' printed. Stack and queues completely empty.", angle: 360 },
  // 21 summary
  { codeLine: 0, stack: [], webapi: [], microtasks: [], macrotasks: [], output: ['1. Start', '2. End', '3. Promise', '3b. qMT', '4. Timeout'], renderActive: false, activeZone: 'idle', note: '✅ Complete cycle verified: Synchronous → Microtasks → Render → Macrotask.', angle: 360 },
  // 22 quiz
  { codeLine: 0, stack: [], webapi: [], microtasks: [], macrotasks: [], output: ['1. Start', '2. End', '3. Promise', '3b. qMT', '4. Timeout'], renderActive: false, activeZone: 'idle', note: '🏆 THE LAW OF JS CONCURRENCY: Microtasks always preempt Macrotasks!', angle: 360 },
]

const cur = computed(() => states[Math.min(s.value, states.length - 1)])

const codeLines = [
  { num: 1, text: "console.log('1. Start');" },
  { num: 2, text: "setTimeout(() => console.log('4. Timeout'), 10);" },
  { num: 3, text: "Promise.resolve().then(() => console.log('3. Promise'));" },
  { num: 4, text: "queueMicrotask(() => console.log('3b. qMT'));" },
  { num: 5, text: "console.log('2. End');" },
]
</script>

<template>
  <div class="h-full flex flex-col justify-between select-none text-slate-800 text-xs">
    <!-- Header -->
    <div class="flex items-center gap-2 pb-1 border-b border-slate-200 shrink-0">
      <span class="text-xl font-black" :class="cur.activeZone !== 'idle' ? 'animate-spin text-amber-600' : ''">↻</span>
      <div>
        <h2 class="text-base font-black text-slate-900 leading-tight">Event Loop — Full Lifecycle Interactive Simulation</h2>
        <p class="text-[10px] text-slate-500">Chapter 3 of 5 · Complete End-to-End Execution Trace</p>
      </div>
      <div class="ml-auto flex items-center gap-1.5">
        <span class="px-2 py-0.5 rounded text-[10px] font-bold border uppercase"
          :class="{
            'bg-sky-100 border-sky-300 text-sky-900': cur.activeZone === 'stack',
            'bg-emerald-100 border-emerald-300 text-emerald-900': cur.activeZone === 'webapi',
            'bg-violet-100 border-violet-300 text-violet-900': cur.activeZone === 'micro',
            'bg-rose-100 border-rose-300 text-rose-900': cur.activeZone === 'render',
            'bg-amber-100 border-amber-300 text-amber-900': cur.activeZone === 'macro',
            'bg-slate-100 border-slate-300 text-slate-700': cur.activeZone === 'idle' || cur.activeZone === 'code',
          }"
        >
          Zone: {{ cur.activeZone }}
        </span>
        <div class="px-2 py-0.5 rounded bg-sky-600 text-white text-[10px] font-bold">Step {{ s }}/22</div>
      </div>
    </div>

    <!-- Active Action Banner -->
    <div
      class="border rounded-lg px-2.5 py-1 text-[11px] font-medium flex items-center gap-1.5 transition-all duration-300 shrink-0"
      :class="{
        'bg-sky-50 border-sky-400 text-sky-900': cur.activeZone === 'stack',
        'bg-emerald-50 border-emerald-400 text-emerald-900': cur.activeZone === 'webapi',
        'bg-violet-50 border-violet-400 text-violet-900': cur.activeZone === 'micro',
        'bg-rose-50 border-rose-400 text-rose-900': cur.activeZone === 'render',
        'bg-amber-50 border-amber-400 text-amber-900': cur.activeZone === 'macro',
        'bg-slate-50 border-slate-300 text-slate-700': cur.activeZone === 'idle' || cur.activeZone === 'code',
      }"
    >
      <span class="animate-pulse font-bold">▶</span>
      <span>{{ cur.note }}</span>
    </div>

    <!-- Main 4-Column Grid -->
    <div class="grid grid-cols-12 gap-2 flex-1 min-h-0 py-1">
      <!-- Col 1: Code & Output (3.5 cols) -->
      <div class="col-span-4 border border-slate-300 rounded-lg p-2 bg-slate-900 text-slate-100 flex flex-col justify-between">
        <div>
          <div class="text-[10px] font-black text-slate-400 uppercase tracking-wider mb-1 flex justify-between">
            <span>📄 Source Code</span>
            <span class="text-[9px] text-amber-400 font-mono">Sync + Async</span>
          </div>
          <div class="font-mono text-[9px] flex flex-col gap-0.5">
            <div
              v-for="line in codeLines" :key="line.num"
              class="px-1.5 py-0.5 rounded flex items-center gap-1.5 transition-all duration-150"
              :class="{
                'bg-sky-500/30 text-sky-200 font-bold border-l-2 border-sky-400': cur.codeLine === line.num,
                'text-slate-400': cur.codeLine !== line.num
              }"
            >
              <span class="text-[8px] text-slate-600 select-none w-3">{{ line.num }}</span>
              <span class="truncate">{{ line.text }}</span>
            </div>
          </div>
        </div>

        <!-- Terminal Output -->
        <div class="bg-slate-800 p-1.5 rounded border border-slate-700 text-[10px] mt-1">
          <div class="text-[8px] text-slate-400 uppercase tracking-wider mb-0.5 flex justify-between">
            <span>Console Output</span>
            <span class="text-emerald-400 font-mono">{{ cur.output.length }}/4</span>
          </div>
          <div v-if="cur.output.length === 0" class="text-slate-500 text-[9px] italic">awaiting output...</div>
          <div v-for="(o, i) in cur.output" :key="i" class="font-mono text-emerald-400 font-bold text-[9px]">
            ▶ "{{ o }}"
          </div>
        </div>
      </div>

      <!-- Col 2: Call Stack (2.5 cols) -->
      <div
        class="col-span-3 border-2 rounded-lg p-2 bg-slate-50 flex flex-col justify-between transition-all duration-200"
        :class="cur.activeZone === 'stack' ? 'border-sky-500 bg-sky-50/50 shadow-xs ring-1 ring-sky-300' : 'border-slate-200'"
      >
        <div class="flex items-center justify-between text-[10px] font-black text-slate-700 uppercase">
          <span>⚡ Call Stack</span>
          <span class="bg-sky-100 text-sky-800 px-1 rounded text-[8px] font-mono">{{ cur.stack.length }} frames</span>
        </div>

        <div class="flex-1 flex flex-col-reverse gap-1 justify-start py-1 min-h-0">
          <div v-if="cur.stack.length === 0" class="flex-1 flex items-center justify-center border-2 border-dashed border-slate-300 rounded-lg">
            <span class="text-[9px] text-slate-400 italic">Stack empty</span>
          </div>
          <div
            v-for="(f, i) in cur.stack" :key="f + i"
            class="rounded px-1.5 py-1 text-[9px] font-bold border flex items-center justify-between shadow-xs transition-all duration-200"
            :class="i === cur.stack.length - 1 ? 'bg-sky-600 text-white border-sky-700 animate-pulse' : 'bg-sky-50 border-sky-300 text-sky-900'"
          >
            <span class="truncate">{{ f }}</span>
            <span v-if="i === cur.stack.length - 1" class="text-[7px] bg-black/20 px-1 rounded ml-1 shrink-0">TOP</span>
          </div>
        </div>

        <div class="text-[8px] text-slate-500 text-center bg-white border border-slate-200 rounded py-0.2">
          LIFO Execution
        </div>
      </div>

      <!-- Col 3: Host Web APIs & Event Loop Hub (2 cols) -->
      <div class="col-span-2 flex flex-col gap-1.5">
        <!-- Web APIs -->
        <div
          class="border-2 rounded-lg p-1.5 flex-1 flex flex-col justify-between transition-all duration-200"
          :class="cur.activeZone === 'webapi' ? 'border-emerald-500 bg-emerald-50 ring-1 ring-emerald-300 shadow-xs' : 'border-slate-200 bg-white'"
        >
          <div class="text-[9px] font-black text-slate-700 uppercase">🌐 Web APIs (Host)</div>
          <div class="flex-1 flex flex-col justify-center">
            <div v-if="cur.webapi.length === 0" class="text-[8px] text-slate-400 italic text-center">Idle</div>
            <div
              v-for="w in cur.webapi" :key="w"
              class="text-[8px] font-bold font-mono rounded p-1 bg-emerald-100 border border-emerald-400 text-emerald-950 text-center animate-pulse"
            >
              {{ w }}
            </div>
          </div>
        </div>

        <!-- Event Loop Wheel -->
        <div class="border border-slate-200 rounded-lg p-1 bg-slate-50 flex flex-col items-center justify-center shrink-0">
          <div
            class="w-10 h-10 rounded-full border-2 flex items-center justify-center shadow-xs transition-all duration-300"
            :class="cur.activeZone !== 'idle' ? 'border-amber-500 bg-amber-100 ring-2 ring-amber-300' : 'border-slate-300 bg-white'"
          >
            <span
              class="text-base font-black text-amber-700 transition-transform duration-300"
              :style="{ transform: `rotate(${cur.angle}deg)` }"
            >↻</span>
          </div>
          <div class="text-[8px] font-black text-slate-700 mt-0.5">EVENT LOOP</div>
        </div>
      </div>

      <!-- Col 4: Queues & Pipeline (3 cols) -->
      <div class="col-span-3 flex flex-col gap-1">
        <!-- Microtask VIP Queue -->
        <div
          class="border-2 rounded-lg p-1.5 flex-1 flex flex-col transition-all duration-200"
          :class="cur.activeZone === 'micro' ? 'border-violet-500 bg-violet-50 ring-1 ring-violet-300 shadow-xs' : 'border-slate-200 bg-white'"
        >
          <div class="flex items-center justify-between text-[9px] font-black text-violet-900 uppercase">
            <span>👑 Microtasks (VIP)</span>
            <span class="bg-violet-600 text-white px-1 rounded text-[7px]">Drain 100%</span>
          </div>
          <div class="flex-1 flex flex-col gap-0.5 justify-start mt-0.5">
            <div v-if="cur.microtasks.length === 0" class="text-[8px] text-slate-400 italic">Drained ✓</div>
            <div
              v-for="m in cur.microtasks" :key="m"
              class="text-[8px] font-bold rounded px-1.5 py-0.5 bg-violet-600 text-white truncate shadow-xs animate-pulse"
            >
              ⭐ {{ m }}
            </div>
          </div>
        </div>

        <!-- Render Pipeline -->
        <div
          class="border-2 rounded-lg p-1 transition-all duration-200 shrink-0"
          :class="cur.renderActive ? 'border-rose-500 bg-rose-50 ring-1 ring-rose-300 shadow-xs' : 'border-slate-200 bg-white'"
        >
          <div class="flex items-center justify-between text-[8px] font-black text-slate-700">
            <span>🎨 Render (60 FPS)</span>
            <span :class="cur.renderActive ? 'text-rose-600 font-bold animate-pulse' : 'text-slate-400'">
              {{ cur.renderActive ? 'V-SYNC ACTIVE' : 'Idle' }}
            </span>
          </div>
          <div class="text-[7px] text-slate-500">rAF → Style → Layout → Paint</div>
        </div>

        <!-- Macrotask Queue -->
        <div
          class="border-2 rounded-lg p-1.5 flex-1 flex flex-col transition-all duration-200"
          :class="cur.activeZone === 'macro' ? 'border-amber-500 bg-amber-50 ring-1 ring-amber-300 shadow-xs' : 'border-slate-200 bg-white'"
        >
          <div class="flex items-center justify-between text-[9px] font-black text-amber-900 uppercase">
            <span>⏱️ Macrotasks</span>
            <span class="bg-amber-600 text-white px-1 rounded text-[7px]">1 Per Turn</span>
          </div>
          <div class="flex-1 flex flex-col gap-0.5 justify-start mt-0.5">
            <div v-if="cur.macrotasks.length === 0" class="text-[8px] text-slate-400 italic">Empty</div>
            <div
              v-for="t in cur.macrotasks" :key="t"
              class="text-[8px] font-bold rounded px-1.5 py-0.5 bg-amber-400 text-amber-950 truncate border border-amber-500 shadow-xs"
            >
              ⏱ {{ t }}
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
