<script setup lang="ts">
import { computed } from 'vue'

const props = defineProps<{ step?: number }>()
const s = computed(() => Math.min(Math.max(props.step ?? 0, 0), 22))

interface QueueSimState {
  codeLine: number
  stack: string[]
  microQueue: string[]
  macroQueue: string[]
  output: string[]
  activeZone: 'code' | 'stack' | 'micro' | 'macro' | 'loop' | 'idle'
  note: string
}

const states: QueueSimState[] = [
  // 0
  { codeLine: 0, stack: [], microQueue: [], macroQueue: [], output: [], activeZone: 'idle', note: '▶ Program ready. Watch how Microtasks take total priority over Macrotasks.' },
  // 1
  { codeLine: 1, stack: ['main()'], microQueue: [], macroQueue: [], output: [], activeZone: 'stack', note: 'Global script starts: main() frame pushed to Call Stack.' },
  // 2
  { codeLine: 2, stack: ['main()', "log('1. Sync Start')"], microQueue: [], macroQueue: [], output: [], activeZone: 'stack', note: "console.log('1. Sync Start') executes synchronously." },
  // 3
  { codeLine: 2, stack: ['main()'], microQueue: [], macroQueue: [], output: ['1. Sync Start'], activeZone: 'stack', note: "✓ '1. Sync Start' logged. Frame popped from stack." },
  // 4
  { codeLine: 3, stack: ['main()', 'setTimeout(T1, 0)'], microQueue: [], macroQueue: [], output: ['1. Sync Start'], activeZone: 'stack', note: 'setTimeout(T1, 0) called. Handed off to Web API timers.' },
  // 5
  { codeLine: 4, stack: ['main()'], microQueue: [], macroQueue: ['T1 (Timer 1)'], output: ['1. Sync Start'], activeZone: 'macro', note: 'Timer 1 expires (0ms) → T1 enters Macrotask Queue.' },
  // 6
  { codeLine: 6, stack: ['main()', 'Promise.resolve().then(P1)'], microQueue: [], macroQueue: ['T1 (Timer 1)'], output: ['1. Sync Start'], activeZone: 'stack', note: 'Promise.resolve() is fulfilled. Enqueues P1 into Microtask VIP queue.' },
  // 7
  { codeLine: 7, stack: ['main()'], microQueue: ['P1 (Micro 1)'], macroQueue: ['T1 (Timer 1)'], output: ['1. Sync Start'], activeZone: 'micro', note: '⭐ P1 enters Microtask Queue. Notice: jumps ahead of T1!' },
  // 8
  { codeLine: 11, stack: ['main()', 'setTimeout(T2, 0)'], microQueue: ['P1 (Micro 1)'], macroQueue: ['T1 (Timer 1)'], output: ['1. Sync Start'], activeZone: 'stack', note: 'setTimeout(T2, 0) called. Handed off to Web APIs.' },
  // 9
  { codeLine: 11, stack: ['main()'], microQueue: ['P1 (Micro 1)'], macroQueue: ['T1 (Timer 1)', 'T2 (Timer 2)'], output: ['1. Sync Start'], activeZone: 'macro', note: 'T2 expires → appended behind T1 in Macrotask queue.' },
  // 10
  { codeLine: 13, stack: ['main()', "log('2. Sync End')"], microQueue: ['P1 (Micro 1)'], macroQueue: ['T1 (Timer 1)', 'T2 (Timer 2)'], output: ['1. Sync Start'], activeZone: 'stack', note: "console.log('2. Sync End') executes synchronously." },
  // 11
  { codeLine: 13, stack: ['main()'], microQueue: ['P1 (Micro 1)'], macroQueue: ['T1 (Timer 1)', 'T2 (Timer 2)'], output: ['1. Sync Start', '2. Sync End'], activeZone: 'stack', note: "✓ '2. Sync End' printed. End of synchronous statements." },
  // 12
  { codeLine: 0, stack: [], microQueue: ['P1 (Micro 1)'], macroQueue: ['T1 (Timer 1)', 'T2 (Timer 2)'], output: ['1. Sync Start', '2. Sync End'], activeZone: 'loop', note: '🔑 main() finishes! Call Stack is EMPTY. Event Loop checks Microtasks FIRST!' },
  // 13
  { codeLine: 7, stack: ['P1()'], microQueue: [], macroQueue: ['T1 (Timer 1)', 'T2 (Timer 2)'], output: ['1. Sync Start', '2. Sync End'], activeZone: 'stack', note: 'Event loop pops P1 from Microtask queue and pushes it to Call Stack.' },
  // 14
  { codeLine: 8, stack: ['P1()', "log('3. Micro 1')"], microQueue: [], macroQueue: ['T1 (Timer 1)', 'T2 (Timer 2)'], output: ['1. Sync Start', '2. Sync End'], activeZone: 'stack', note: "P1 prints '3. Micro 1'. Chained .then() queues P2 into Microtask queue!" },
  // 15
  { codeLine: 9, stack: [], microQueue: ['P2 (Chained Micro)'], macroQueue: ['T1 (Timer 1)', 'T2 (Timer 2)'], output: ['1. Sync Start', '2. Sync End', '3. Micro 1'], activeZone: 'micro', note: 'P1 finishes. Microtask queue still has P2! Event loop MUST drain it first!' },
  // 16
  { codeLine: 10, stack: ['P2()'], microQueue: [], macroQueue: ['T1 (Timer 1)', 'T2 (Timer 2)'], output: ['1. Sync Start', '2. Sync End', '3. Micro 1'], activeZone: 'stack', note: 'P2 pushed to Call Stack before ANY macrotask can execute.' },
  // 17
  { codeLine: 10, stack: [], microQueue: [], macroQueue: ['T1 (Timer 1)', 'T2 (Timer 2)'], output: ['1. Sync Start', '2. Sync End', '3. Micro 1', '4. Micro 2'], activeZone: 'loop', note: "✓ '4. Micro 2' logged. Microtask queue is now 100% EMPTY!" },
  // 18
  { codeLine: 3, stack: ['T1()'], microQueue: [], macroQueue: ['T2 (Timer 2)'], output: ['1. Sync Start', '2. Sync End', '3. Micro 1', '4. Micro 2'], activeZone: 'stack', note: 'Event Loop picks ONE macrotask: Dequeues T1 onto Call Stack.' },
  // 19
  { codeLine: 4, stack: ['T1()', "log('5. Macro Timer 1')"], microQueue: [], macroQueue: ['T2 (Timer 2)'], output: ['1. Sync Start', '2. Sync End', '3. Micro 1', '4. Micro 2'], activeZone: 'stack', note: "T1 logs '5. Macro Timer 1'. Then schedules a new microtask P3!" },
  // 20
  { codeLine: 5, stack: ['P3()'], microQueue: [], macroQueue: ['T2 (Timer 2)'], output: ['1. Sync Start', '2. Sync End', '3. Micro 1', '4. Micro 2', '5. Macro Timer 1'], activeZone: 'micro', note: 'T1 finishes. Event loop drains P3 IMMEDIATELY before running T2!' },
  // 21
  { codeLine: 11, stack: ['T2()'], microQueue: [], macroQueue: [], output: ['1. Sync Start', '2. Sync End', '3. Micro 1', '4. Micro 2', '5. Macro Timer 1', '6. Micro in Timer'], activeZone: 'stack', note: "P3 logs '6. Micro in Timer'. Now Event Loop picks next macrotask: T2!" },
  // 22
  { codeLine: 0, stack: [], microQueue: [], macroQueue: [], output: ['1. Sync Start', '2. Sync End', '3. Micro 1', '4. Micro 2', '5. Macro Timer 1', '6. Micro in Timer', '7. Macro Timer 2'], activeZone: 'idle', note: '🏆 ALL QUEUES EMPTY! Full execution order: Sync → Microtasks → Macrotask 1 → Nested Micro → Macrotask 2.' },
]

const cur = computed(() => states[Math.min(s.value, states.length - 1)])

const codeLines = [
  { num: 1, text: "console.log('1. Sync Start');" },
  { num: 2, text: "" },
  { num: 3, text: "setTimeout(() => {" },
  { num: 4, text: "  console.log('5. Macro Timer 1');" },
  { num: 5, text: "  Promise.resolve().then(() => console.log('6. Micro in Timer'));" },
  { num: 6, text: "}, 0);" },
  { num: 7, text: "" },
  { num: 8, text: "Promise.resolve().then(() => {" },
  { num: 9, text: "  console.log('3. Micro 1');" },
  { num: 10, text: "}).then(() => console.log('4. Micro 2'));" },
  { num: 11, text: "" },
  { num: 12, text: "setTimeout(() => console.log('7. Macro Timer 2'), 0);" },
  { num: 13, text: "console.log('2. Sync End');" },
]
</script>

<template>
  <div class="h-full flex flex-col gap-1.5 select-none text-slate-800">
    <!-- Header -->
    <div class="flex items-center gap-3 pb-1 border-b-2 border-slate-200">
      <span class="text-2xl">⚡</span>
      <div>
        <h2 class="text-xl font-black text-slate-900 leading-tight">Micro vs Macro Live Execution Trace</h2>
        <p class="text-xs text-slate-500">Chapter 5 of 6 · Interactive Simulator</p>
      </div>
      <div class="ml-auto px-2 py-1 rounded-lg bg-sky-600 text-white text-xs font-bold">Step {{ s }}/22</div>
    </div>

    <!-- Active Note Banner -->
    <div
      class="rounded-xl px-3 py-1.5 text-xs font-bold border-2 flex items-center gap-2 transition-all duration-300"
      :class="{
        'bg-sky-50 border-sky-400 text-sky-900': cur.activeZone === 'stack',
        'bg-violet-50 border-violet-400 text-violet-900': cur.activeZone === 'micro',
        'bg-amber-50 border-amber-400 text-amber-900': cur.activeZone === 'macro',
        'bg-emerald-50 border-emerald-400 text-emerald-900': cur.activeZone === 'loop',
        'bg-slate-50 border-slate-300 text-slate-700': cur.activeZone === 'idle',
      }"
    >
      <span class="animate-pulse">▶</span>
      <span>{{ cur.note }}</span>
    </div>

    <!-- 4-Column Grid -->
    <div class="grid grid-cols-12 gap-2 flex-1 min-h-0">
      <!-- Col 1: Code (4 cols) -->
      <div class="col-span-4 border-2 border-slate-200 rounded-xl p-2 bg-slate-900 text-slate-100 flex flex-col justify-between overflow-hidden">
        <div>
          <div class="text-[10px] font-black text-slate-400 uppercase tracking-wider mb-1 flex items-center justify-between">
            <span>📄 Program Under Test</span>
            <span class="text-sky-400 text-[9px] font-mono">2 Timers + 2 Promises</span>
          </div>
          <div class="font-mono text-[10px] leading-tight flex flex-col gap-0.5">
            <div
              v-for="line in codeLines" :key="line.num"
              class="px-1.5 py-0.5 rounded transition-all duration-200 flex items-center gap-1.5"
              :class="{
                'bg-sky-500/30 text-sky-200 border-l-2 border-sky-400 font-bold': cur.codeLine === line.num,
                'text-slate-500': cur.codeLine !== line.num && line.text,
                'opacity-0': !line.text
              }"
            >
              <span class="text-[8px] text-slate-600 select-none w-3 shrink-0">{{ line.num }}</span>
              <span class="truncate">{{ line.text }}</span>
            </div>
          </div>
        </div>

        <div class="bg-slate-800 p-1.5 rounded-lg border border-slate-700 text-[9px] text-slate-300">
          <span class="text-violet-400 font-bold">Rule:</span> Microtasks run between stack frames & always before next macrotask.
        </div>
      </div>

      <!-- Col 2: Call Stack (2.5 cols) -->
      <div
        class="col-span-3 border-2 rounded-xl p-2.5 flex flex-col transition-all duration-300"
        :class="cur.activeZone === 'stack' ? 'border-sky-500 bg-sky-50/50 shadow-md ring-2 ring-sky-200' : 'border-slate-200 bg-white'"
      >
        <div class="text-[10px] font-black text-slate-700 uppercase tracking-wider mb-1 flex items-center justify-between">
          <span>⚡ Call Stack</span>
          <span class="bg-sky-100 text-sky-800 px-1.5 py-0.5 rounded text-[9px] font-bold">{{ cur.stack.length }} frames</span>
        </div>
        <div class="text-[8px] text-slate-400 mb-2">V8 Single Thread Engine</div>

        <div class="flex-1 flex flex-col-reverse gap-1.5 justify-start">
          <div v-if="cur.stack.length === 0" class="flex items-center justify-center h-28 border-2 border-dashed border-slate-200 rounded-lg">
            <span class="text-[10px] text-slate-400 italic">Stack empty</span>
          </div>
          <div
            v-for="(f, i) in cur.stack" :key="f + i"
            class="rounded-lg px-2 py-1 text-[10px] font-bold border transition-all duration-300 shadow-sm"
            :class="i === cur.stack.length - 1 ? 'bg-sky-600 text-white border-sky-700 animate-pulse' : 'bg-sky-50 border-sky-300 text-sky-900'"
          >
            <div class="flex items-center justify-between">
              <span class="truncate">{{ f }}</span>
              <span class="text-[8px] px-1 rounded bg-black/20 shrink-0 ml-1">{{ i === cur.stack.length - 1 ? 'TOP' : '' }}</span>
            </div>
          </div>
        </div>
      </div>

      <!-- Col 3: Two Queues (3 cols) -->
      <div class="col-span-3 flex flex-col gap-2">
        <!-- Microtask VIP Queue -->
        <div
          class="border-2 rounded-xl p-2 flex-1 flex flex-col transition-all duration-300"
          :class="cur.activeZone === 'micro' ? 'border-violet-500 bg-violet-50/70 shadow-md ring-2 ring-violet-200' : 'border-slate-200 bg-white'"
        >
          <div class="text-[10px] font-black text-violet-900 uppercase tracking-wider mb-1 flex items-center justify-between">
            <span class="flex items-center gap-1">👑 Microtasks (VIP)</span>
            <span class="bg-violet-600 text-white px-1.5 py-0.2 rounded text-[8px] font-bold">DRAIN ALL</span>
          </div>

          <div class="flex-1 flex flex-col gap-1 justify-start">
            <div v-if="cur.microQueue.length === 0" class="flex items-center justify-center h-12 border-2 border-dashed border-slate-200 rounded-lg">
              <span class="text-[8px] text-slate-400 italic">Queue drained ✓</span>
            </div>
            <div
              v-for="m in cur.microQueue" :key="m"
              class="rounded-md px-2 py-1 text-[10px] font-bold bg-violet-600 text-white flex items-center justify-between shadow-sm animate-pulse"
            >
              <span>⭐ {{ m }}</span>
              <span class="text-[8px] bg-white/20 px-1 rounded">VIP</span>
            </div>
          </div>
        </div>

        <!-- Macrotask Queue -->
        <div
          class="border-2 rounded-xl p-2 flex-1 flex flex-col transition-all duration-300"
          :class="cur.activeZone === 'macro' ? 'border-amber-500 bg-amber-50/70 shadow-md ring-2 ring-amber-200' : 'border-slate-200 bg-white'"
        >
          <div class="text-[10px] font-black text-amber-900 uppercase tracking-wider mb-1 flex items-center justify-between">
            <span class="flex items-center gap-1">⏱️ Macrotasks</span>
            <span class="bg-amber-600 text-white px-1.5 py-0.2 rounded text-[8px] font-bold">1 PER TURN</span>
          </div>

          <div class="flex-1 flex flex-col gap-1 justify-start">
            <div v-if="cur.macroQueue.length === 0" class="flex items-center justify-center h-12 border-2 border-dashed border-slate-200 rounded-lg">
              <span class="text-[8px] text-slate-400 italic">Queue empty</span>
            </div>
            <div
              v-for="t in cur.macroQueue" :key="t"
              class="rounded-md px-2 py-1 text-[10px] font-bold bg-amber-400 text-amber-950 border border-amber-500 flex items-center justify-between shadow-sm"
            >
              <span>⏱ {{ t }}</span>
              <span class="text-[8px] bg-amber-600/30 px-1 rounded">task</span>
            </div>
          </div>
        </div>
      </div>

      <!-- Col 4: Output Terminal (2 cols) -->
      <div class="col-span-2 border-2 border-slate-300 rounded-xl p-2 bg-slate-950 text-slate-100 flex flex-col">
        <div class="text-[10px] font-black text-slate-400 uppercase tracking-wider mb-1 flex items-center justify-between">
          <span>📟 Output</span>
          <span class="text-emerald-400 text-[9px] font-mono">{{ cur.output.length }}/7</span>
        </div>
        <div class="font-mono text-[9px] flex-1 flex flex-col gap-1 overflow-hidden">
          <div v-if="cur.output.length === 0" class="text-slate-600 italic text-[9px]">Awaiting output...</div>
          <div
            v-for="(out, idx) in cur.output" :key="idx"
            class="text-emerald-400 flex items-start gap-1 leading-tight"
          >
            <span class="text-slate-600 select-none">></span>
            <span class="font-bold">{{ out }}</span>
          </div>
        </div>
        <div class="border-t border-slate-800 pt-1 mt-1 text-[8px] text-slate-500 font-mono">
          Final order is deterministic!
        </div>
      </div>
    </div>
  </div>
</template>
