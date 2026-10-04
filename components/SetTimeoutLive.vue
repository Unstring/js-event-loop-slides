<script setup lang="ts">
import { computed } from 'vue'

const props = defineProps<{ step?: number }>()
const s = computed(() => Math.min(Math.max(props.step ?? 0, 0), 22))

interface TimeoutSimState {
  codeLine: number
  stack: string[]
  timerState: 'idle' | 'registered' | 'ticking' | 'expired'
  timerMs: string
  macroQueue: string[]
  output: string[]
  activeZone: 'code' | 'stack' | 'timer' | 'macro' | 'loop' | 'idle'
  elapsedMs: number
  note: string
}

const states: TimeoutSimState[] = [
  // 0
  { codeLine: 0, stack: [], timerState: 'idle', timerMs: '--', macroQueue: [], output: [], activeZone: 'idle', elapsedMs: 0, note: '▶ Program loaded. Ready to trace setTimeout(fn, 0) internals.' },
  // 1
  { codeLine: 1, stack: ['main()'], timerState: 'idle', timerMs: '--', macroQueue: [], output: [], activeZone: 'stack', elapsedMs: 0, note: 'Global script begins: main() pushed to Call Stack.' },
  // 2
  { codeLine: 2, stack: ['main()', "console.log('1. Start')"], timerState: 'idle', timerMs: '--', macroQueue: [], output: [], activeZone: 'stack', elapsedMs: 1, note: "console.log('1. Start') executes synchronously on Call Stack." },
  // 3
  { codeLine: 2, stack: ['main()'], timerState: 'idle', timerMs: '--', macroQueue: [], output: ['1. Start'], activeZone: 'stack', elapsedMs: 2, note: "✓ '1. Start' printed to console. Stack frame popped." },
  // 4
  { codeLine: 4, stack: ['main()', 'setTimeout(cb, 0)'], timerState: 'idle', timerMs: '--', macroQueue: [], output: ['1. Start'], activeZone: 'stack', elapsedMs: 3, note: 'setTimeout(cb, 0) invoked! JS engine (V8) passes cb to Host Web API.' },
  // 5
  { codeLine: 4, stack: ['main()'], timerState: 'registered', timerMs: '0ms', macroQueue: [], output: ['1. Start'], activeZone: 'timer', elapsedMs: 4, note: 'Timer registered in Browser Min-Heap. setTimeout frame POPS immediately!' },
  // 6
  { codeLine: 5, stack: ['main()', 'blockCpuFor(1000)'], timerState: 'ticking', timerMs: '0ms', macroQueue: [], output: ['1. Start'], activeZone: 'stack', elapsedMs: 5, note: 'Call Stack begins heavy computation blockCpuFor(1000ms). Stack is BUSY!' },
  // 7
  { codeLine: 5, stack: ['main()', 'blockCpuFor(1000) [T: 100ms]'], timerState: 'expired', timerMs: 'EXPIRED (0ms)', macroQueue: ['cb()'], output: ['1. Start'], activeZone: 'macro', elapsedMs: 100, note: '⏱ Timer expires at T+1ms! Web API moves cb() to Macrotask Queue.' },
  // 8
  { codeLine: 5, stack: ['main()', 'blockCpuFor(1000) [T: 300ms]'], timerState: 'expired', timerMs: 'Done', macroQueue: ['cb()'], output: ['1. Start'], activeZone: 'loop', elapsedMs: 300, note: 'Event Loop checks: "Is Call Stack empty?" NO! blockCpu is still running.' },
  // 9
  { codeLine: 5, stack: ['main()', 'blockCpuFor(1000) [T: 600ms]'], timerState: 'expired', timerMs: 'Done', macroQueue: ['cb()'], output: ['1. Start'], activeZone: 'macro', elapsedMs: 600, note: 'cb() must WAIT in Macrotask queue. JavaScript thread CANNOT be interrupted!' },
  // 10
  { codeLine: 5, stack: ['main()', 'blockCpuFor(1000) [T: 900ms]'], timerState: 'expired', timerMs: 'Done', macroQueue: ['cb()'], output: ['1. Start'], activeZone: 'macro', elapsedMs: 900, note: 'Wait continues... Latency gap is now +900ms over the specified 0ms!' },
  // 11
  { codeLine: 5, stack: ['main()'], timerState: 'expired', timerMs: 'Done', macroQueue: ['cb()'], output: ['1. Start'], activeZone: 'stack', elapsedMs: 1000, note: 'blockCpuFor(1000) FINALLY finishes after 1000ms and pops off stack.' },
  // 12
  { codeLine: 6, stack: ['main()', "console.log('2. End')"], timerState: 'expired', timerMs: 'Done', macroQueue: ['cb()'], output: ['1. Start'], activeZone: 'stack', elapsedMs: 1001, note: "console.log('2. End') pushed to stack. Still synchronous code!" },
  // 13
  { codeLine: 6, stack: ['main()'], timerState: 'expired', timerMs: 'Done', macroQueue: ['cb()'], output: ['1. Start', '2. End'], activeZone: 'stack', elapsedMs: 1002, note: "✓ '2. End' printed to console. Frame popped." },
  // 14
  { codeLine: 7, stack: [], timerState: 'expired', timerMs: 'Done', macroQueue: ['cb()'], output: ['1. Start', '2. End'], activeZone: 'loop', elapsedMs: 1003, note: '🔑 main() finishes! Call Stack is now completely EMPTY for the first time!' },
  // 15
  { codeLine: 7, stack: [], timerState: 'expired', timerMs: 'Done', macroQueue: ['cb()'], output: ['1. Start', '2. End'], activeZone: 'loop', elapsedMs: 1003, note: 'Event Loop wakes up: Checks Microtasks (0 pending) → Checks Render (none).' },
  // 16
  { codeLine: 7, stack: [], timerState: 'expired', timerMs: 'Done', macroQueue: ['cb()'], output: ['1. Start', '2. End'], activeZone: 'macro', elapsedMs: 1004, note: 'Event Loop checks Macrotask Queue: Finds cb() waiting since T+1ms!' },
  // 17
  { codeLine: 3, stack: ['cb()'], timerState: 'expired', timerMs: 'Done', macroQueue: [], output: ['1. Start', '2. End'], activeZone: 'stack', elapsedMs: 1005, note: 'Event Loop dequeues cb() and pushes it onto the Call Stack.' },
  // 18
  { codeLine: 3, stack: ['cb()', "console.log('3. Timeout 0ms')"], timerState: 'expired', timerMs: 'Done', macroQueue: [], output: ['1. Start', '2. End'], activeZone: 'stack', elapsedMs: 1006, note: "cb() executes console.log('3. Timeout 0ms')." },
  // 19
  { codeLine: 3, stack: ['cb()'], timerState: 'expired', timerMs: 'Done', macroQueue: [], output: ['1. Start', '2. End', '3. Timeout 0ms'], activeZone: 'stack', elapsedMs: 1007, note: "✓ '3. Timeout 0ms' printed to console! Frame popped." },
  // 20
  { codeLine: 0, stack: [], timerState: 'expired', timerMs: 'Done', macroQueue: [], output: ['1. Start', '2. End', '3. Timeout 0ms'], activeZone: 'idle', elapsedMs: 1008, note: 'cb() completes and pops. Call Stack is empty again.' },
  // 21
  { codeLine: 0, stack: [], timerState: 'idle', timerMs: '--', macroQueue: [], output: ['1. Start', '2. End', '3. Timeout 0ms'], activeZone: 'idle', elapsedMs: 1008, note: '⏱ RESULT: Specified delay = 0ms. ACTUAL execution time = 1005ms!' },
  // 22
  { codeLine: 0, stack: [], timerState: 'idle', timerMs: '--', macroQueue: [], output: ['1. Start', '2. End', '3. Timeout 0ms'], activeZone: 'idle', elapsedMs: 1008, note: '💡 GOLDEN RULE: setTimeout specifies minimum delay before queueing, NOT execution time!' },
]

const cur = computed(() => states[Math.min(s.value, states.length - 1)])

const codeLines = [
  { num: 1, text: "// Script starts" },
  { num: 2, text: "console.log('1. Start');" },
  { num: 3, text: "setTimeout(() => {" },
  { num: 4, text: "  console.log('3. Timeout 0ms');" },
  { num: 5, text: "}, 0);" },
  { num: 6, text: "blockCpuFor(1000); // 1 sec busy" },
  { num: 7, text: "console.log('2. End');" },
]
</script>

<template>
  <div class="h-full flex flex-col gap-1.5 select-none text-slate-800">
    <!-- Header -->
    <div class="flex items-center gap-3 pb-1 border-b-2 border-slate-200">
      <span class="text-2xl">⏳</span>
      <div>
        <h2 class="text-xl font-black text-slate-900 leading-tight">setTimeout Live: The Latency Gap</h2>
        <p class="text-xs text-slate-500">Chapter 4 of 5 · Interactive Simulator</p>
      </div>
      <div class="ml-auto flex items-center gap-2">
        <span class="px-2 py-0.5 rounded bg-amber-100 border border-amber-300 text-amber-900 font-mono text-xs font-bold">
          Elapsed: {{ cur.elapsedMs }}ms
        </span>
        <div class="px-2 py-1 rounded-lg bg-sky-600 text-white text-xs font-bold">Step {{ s }}/22</div>
      </div>
    </div>

    <!-- Note banner -->
    <div
      class="rounded-xl px-3 py-1.5 text-xs font-bold border-2 flex items-center gap-2 transition-all duration-300"
      :class="{
        'bg-sky-50 border-sky-400 text-sky-900': cur.activeZone === 'stack',
        'bg-emerald-50 border-emerald-400 text-emerald-900': cur.activeZone === 'timer',
        'bg-amber-50 border-amber-400 text-amber-900': cur.activeZone === 'macro',
        'bg-violet-50 border-violet-400 text-violet-900': cur.activeZone === 'loop',
        'bg-slate-50 border-slate-300 text-slate-700': cur.activeZone === 'idle',
      }"
    >
      <span class="animate-pulse">▶</span>
      <span>{{ cur.note }}</span>
    </div>

    <!-- Main 4-column layout -->
    <div class="grid grid-cols-12 gap-2 flex-1 min-h-0">
      <!-- Col 1: Code (4 cols) -->
      <div class="col-span-4 border-2 border-slate-200 rounded-xl p-2.5 bg-slate-900 text-slate-100 flex flex-col justify-between">
        <div>
          <div class="text-[10px] font-black text-slate-400 uppercase tracking-wider mb-1.5 flex items-center justify-between">
            <span>📄 Source Code</span>
            <span class="text-[9px] text-amber-400 font-mono">0ms specified</span>
          </div>
          <div class="font-mono text-[11px] flex flex-col gap-0.5">
            <div
              v-for="line in codeLines" :key="line.num"
              class="px-2 py-0.5 rounded transition-all duration-200 flex items-center gap-2"
              :class="{
                'bg-sky-500/30 text-sky-200 border-l-2 border-sky-400 font-bold': cur.codeLine === line.num,
                'text-slate-400': cur.codeLine !== line.num
              }"
            >
              <span class="text-[9px] text-slate-600 select-none w-3">{{ line.num }}</span>
              <span class="truncate">{{ line.text }}</span>
            </div>
          </div>
        </div>

        <!-- Latency gauge -->
        <div class="bg-slate-800/80 border border-slate-700 rounded-lg p-2 mt-2">
          <div class="flex justify-between text-[10px] mb-1">
            <span class="text-slate-400">Delay Gap Gauge:</span>
            <span class="font-mono font-bold text-amber-400">{{ cur.elapsedMs }}ms</span>
          </div>
          <div class="w-full bg-slate-700 h-2 rounded-full overflow-hidden">
            <div
              class="bg-gradient-to-r from-emerald-400 via-amber-400 to-rose-500 h-full transition-all duration-300"
              :style="{ width: `${Math.min((cur.elapsedMs / 1000) * 100, 100)}%` }"
            ></div>
          </div>
          <div class="flex justify-between text-[8px] text-slate-500 mt-1">
            <span>Target: 0ms</span>
            <span>Actual: {{ cur.elapsedMs }}ms</span>
          </div>
        </div>
      </div>

      <!-- Col 2: Call Stack (3 cols) -->
      <div
        class="col-span-3 border-2 rounded-xl p-2.5 flex flex-col transition-all duration-300"
        :class="cur.activeZone === 'stack' ? 'border-sky-500 bg-sky-50/50 shadow-md ring-2 ring-sky-200' : 'border-slate-200 bg-white'"
      >
        <div class="text-[10px] font-black text-slate-700 uppercase tracking-wider mb-1 flex items-center justify-between">
          <span>⚡ Call Stack</span>
          <span class="bg-sky-100 text-sky-800 px-1.5 py-0.5 rounded text-[9px] font-bold">{{ cur.stack.length }} frames</span>
        </div>
        <div class="text-[9px] text-slate-400 mb-2">V8 Single Thread (LIFO)</div>

        <div class="flex-1 flex flex-col-reverse gap-1.5 justify-start">
          <div v-if="cur.stack.length === 0" class="flex items-center justify-center h-28 border-2 border-dashed border-slate-200 rounded-lg">
            <span class="text-[10px] text-slate-400 italic">Stack empty (idle)</span>
          </div>
          <div
            v-for="(f, i) in cur.stack" :key="f + i"
            class="rounded-lg px-2 py-1.5 text-[10px] font-bold border transition-all duration-300 shadow-sm"
            :class="i === cur.stack.length - 1 ? 'bg-sky-600 text-white border-sky-700 animate-pulse' : 'bg-sky-50 border-sky-300 text-sky-900'"
          >
            <div class="flex items-center justify-between">
              <span class="truncate">{{ f }}</span>
              <span class="text-[8px] px-1 rounded bg-black/20 shrink-0 ml-1">{{ i === cur.stack.length - 1 ? 'TOP' : '' }}</span>
            </div>
          </div>
        </div>
      </div>

      <!-- Col 3: Host Web APIs (2 cols) & Event Loop -->
      <div class="col-span-2 flex flex-col gap-2">
        <!-- Host Timer -->
        <div
          class="border-2 rounded-xl p-2 flex-1 flex flex-col transition-all duration-300"
          :class="cur.activeZone === 'timer' ? 'border-emerald-500 bg-emerald-50/60 ring-2 ring-emerald-200 shadow-md' : 'border-slate-200 bg-white'"
        >
          <div class="text-[10px] font-black text-slate-700 uppercase tracking-wider mb-1">🌐 Web APIs</div>
          <div class="text-[8px] text-slate-400 mb-1">Host OS Min-Heap</div>

          <div class="flex-1 flex flex-col justify-center items-center">
            <div
              class="w-full text-center p-2 rounded-lg border-2 transition-all duration-300"
              :class="{
                'bg-slate-50 border-slate-200 text-slate-400': cur.timerState === 'idle',
                'bg-amber-100 border-amber-400 text-amber-900 animate-pulse': cur.timerState === 'registered' || cur.timerState === 'ticking',
                'bg-emerald-100 border-emerald-400 text-emerald-900 font-bold': cur.timerState === 'expired',
              }"
            >
              <div class="text-base mb-0.5">⏱</div>
              <div class="text-[10px] font-bold uppercase">{{ cur.timerState }}</div>
              <div class="text-[9px] font-mono mt-0.5">{{ cur.timerMs }}</div>
            </div>
          </div>
        </div>

        <!-- Event Loop Indicator -->
        <div
          class="border-2 rounded-xl p-2 text-center transition-all duration-300"
          :class="cur.activeZone === 'loop' ? 'border-violet-500 bg-violet-50 ring-2 ring-violet-200 shadow-md' : 'border-slate-200 bg-white'"
        >
          <div class="text-[9px] font-black text-slate-600 mb-0.5">EVENT LOOP</div>
          <div class="text-lg transition-transform duration-500" :class="cur.activeZone === 'loop' ? 'animate-spin text-violet-600' : 'text-slate-400'">↻</div>
          <div class="text-[8px] text-slate-500 mt-0.5">
            {{ cur.stack.length > 0 ? 'Waiting for Stack...' : (cur.macroQueue.length > 0 ? 'Pumping task!' : 'Watching') }}
          </div>
        </div>
      </div>

      <!-- Col 4: Macrotask Queue & Console (3 cols) -->
      <div class="col-span-3 flex flex-col gap-2">
        <!-- Macrotask Queue -->
        <div
          class="border-2 rounded-xl p-2.5 flex-1 flex flex-col transition-all duration-300"
          :class="cur.activeZone === 'macro' ? 'border-amber-500 bg-amber-50/60 ring-2 ring-amber-200 shadow-md' : 'border-slate-200 bg-white'"
        >
          <div class="text-[10px] font-black text-slate-700 uppercase tracking-wider mb-1 flex items-center justify-between">
            <span>📬 Macrotask Queue</span>
            <span class="bg-amber-100 text-amber-800 px-1 rounded text-[9px] font-bold">{{ cur.macroQueue.length }} task</span>
          </div>
          <div class="text-[8px] text-slate-400 mb-2">Timer callbacks queue here</div>

          <div class="flex-1 flex flex-col gap-1 justify-start">
            <div v-if="cur.macroQueue.length === 0" class="flex items-center justify-center h-16 border-2 border-dashed border-slate-200 rounded-lg">
              <span class="text-[9px] text-slate-400 italic">Queue empty</span>
            </div>
            <div
              v-for="task in cur.macroQueue" :key="task"
              class="rounded-lg px-2 py-1.5 text-[10px] font-bold bg-amber-400 text-amber-950 border border-amber-500 flex items-center justify-between shadow-sm animate-bounce"
            >
              <span>⏱ {{ task }}</span>
              <span class="text-[8px] bg-amber-600/30 text-amber-950 px-1 rounded">waiting</span>
            </div>
          </div>
        </div>

        <!-- Console Output -->
        <div class="border-2 border-slate-300 rounded-xl p-2.5 bg-slate-950 text-slate-100">
          <div class="text-[10px] font-black text-slate-400 uppercase tracking-wider mb-1 flex items-center justify-between">
            <span>📟 Terminal Output</span>
            <span class="text-emerald-400 text-[9px] font-mono">{{ cur.output.length }} logged</span>
          </div>
          <div class="font-mono text-[10px] min-h-[50px] flex flex-col gap-0.5">
            <div v-if="cur.output.length === 0" class="text-slate-600 italic text-[9px]">Awaiting console.log...</div>
            <div
              v-for="(out, idx) in cur.output" :key="idx"
              class="text-emerald-400 flex items-center gap-1.5"
            >
              <span class="text-slate-600 select-none">></span>
              <span class="font-bold">"{{ out }}"</span>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
