<script setup lang="ts">
import { computed } from 'vue'

const props = defineProps<{ step?: number }>()
const s = computed(() => Math.min(Math.max(props.step ?? 0, 0), 22))

interface AsyncSimState {
  codeLine: number
  stack: string[]
  heapSaved: string | null
  microQueue: string[]
  macroQueue: string[]
  output: string[]
  activeZone: 'code' | 'stack' | 'heap' | 'micro' | 'macro' | 'loop' | 'idle'
  note: string
}

const states: AsyncSimState[] = [
  // 0
  { codeLine: 0, stack: [], heapSaved: null, microQueue: [], macroQueue: [], output: [], activeZone: 'idle', note: '▶ Ready. Tracing async/await suspension & microtask resumption.' },
  // 1
  { codeLine: 1, stack: ['main()'], heapSaved: null, microQueue: [], macroQueue: [], output: [], activeZone: 'stack', note: 'Global script begins: main() frame pushed to Call Stack.' },
  // 2
  { codeLine: 7, stack: ['main()', "log('1. Script Start')"], heapSaved: null, microQueue: [], macroQueue: [], output: [], activeZone: 'stack', note: "Line 7: console.log('1. Script Start') executes synchronously." },
  // 3
  { codeLine: 7, stack: ['main()'], heapSaved: null, microQueue: [], macroQueue: [], output: ['1. Script Start'], activeZone: 'stack', note: "✓ '1. Script Start' logged. Stack frame popped." },
  // 4
  { codeLine: 8, stack: ['main()', 'setTimeout(cb, 0)'], heapSaved: null, microQueue: [], macroQueue: [], output: ['1. Script Start'], activeZone: 'stack', note: 'Line 8: setTimeout(cb, 0) scheduled with Host Web APIs.' },
  // 5
  { codeLine: 8, stack: ['main()'], heapSaved: null, microQueue: [], macroQueue: ['cb: Timeout'], output: ['1. Script Start'], activeZone: 'macro', note: 'Timer expires immediately → cb pushed to Macrotask Queue.' },
  // 6
  { codeLine: 9, stack: ['main()', 'asyncFn()'], heapSaved: null, microQueue: [], macroQueue: ['cb: Timeout'], output: ['1. Script Start'], activeZone: 'stack', note: 'Line 9: asyncFn() invoked! Pushes asyncFn frame onto Call Stack.' },
  // 7
  { codeLine: 2, stack: ['main()', 'asyncFn()', "log('2. asyncFn Start')"], heapSaved: null, microQueue: [], macroQueue: ['cb: Timeout'], output: ['1. Script Start'], activeZone: 'stack', note: '⭐ SURPRISE: Async functions execute SYNCHRONOUSLY until the first await!' },
  // 8
  { codeLine: 2, stack: ['main()', 'asyncFn()'], heapSaved: null, microQueue: [], macroQueue: ['cb: Timeout'], output: ['1. Script Start', '2. asyncFn Start'], activeZone: 'stack', note: "✓ '2. asyncFn Start' logged to console." },
  // 9
  { codeLine: 3, stack: ['main()', 'asyncFn() [await null]'], heapSaved: null, microQueue: [], macroQueue: ['cb: Timeout'], output: ['1. Script Start', '2. asyncFn Start'], activeZone: 'code', note: 'Line 3: Hits await null! V8 converts expression to Promise.resolve(null).' },
  // 10
  { codeLine: 3, stack: ['main()'], heapSaved: 'asyncFn Context (pc: line 4)', microQueue: ['Resume asyncFn()'], macroQueue: ['cb: Timeout'], output: ['1. Script Start', '2. asyncFn Start'], activeZone: 'heap', note: '⏸️ SUSPEND: asyncFn stack pops! State saved to Heap. Resumption scheduled in Microtasks!' },
  // 11
  { codeLine: 10, stack: ['main()', "log('3. Script End')"], heapSaved: 'asyncFn Context (pc: line 4)', microQueue: ['Resume asyncFn()'], macroQueue: ['cb: Timeout'], output: ['1. Script Start', '2. asyncFn Start'], activeZone: 'stack', note: 'Caller continues immediately! main() executes console.log(\'3. Script End\').' },
  // 12
  { codeLine: 10, stack: ['main()'], heapSaved: 'asyncFn Context (pc: line 4)', microQueue: ['Resume asyncFn()'], macroQueue: ['cb: Timeout'], output: ['1. Script Start', '2. asyncFn Start', '3. Script End'], activeZone: 'stack', note: "✓ '3. Script End' printed. Call frame popped." },
  // 13
  { codeLine: 0, stack: [], heapSaved: 'asyncFn Context (pc: line 4)', microQueue: ['Resume asyncFn()'], macroQueue: ['cb: Timeout'], output: ['1. Script Start', '2. asyncFn Start', '3. Script End'], activeZone: 'loop', note: '🔑 main() finishes! Call Stack is completely EMPTY!' },
  // 14
  { codeLine: 0, stack: [], heapSaved: 'asyncFn Context (pc: line 4)', microQueue: ['Resume asyncFn()'], macroQueue: ['cb: Timeout'], output: ['1. Script Start', '2. asyncFn Start', '3. Script End'], activeZone: 'micro', note: 'Event Loop wakes: Drains Microtask Queue FIRST (before checking timers).' },
  // 15
  { codeLine: 4, stack: ['asyncFn() [Resumed]'], heapSaved: null, microQueue: [], macroQueue: ['cb: Timeout'], output: ['1. Script Start', '2. asyncFn Start', '3. Script End'], activeZone: 'stack', note: '▶️ RESUME: V8 restores asyncFn context from Heap back onto Call Stack at line 4!' },
  // 16
  { codeLine: 4, stack: ['asyncFn()', "log('4. asyncFn Resume')"], heapSaved: null, microQueue: [], macroQueue: ['cb: Timeout'], output: ['1. Script Start', '2. asyncFn Start', '3. Script End'], activeZone: 'stack', note: "Executes console.log('4. asyncFn Resume')." },
  // 17
  { codeLine: 5, stack: ['asyncFn()'], heapSaved: null, microQueue: [], macroQueue: ['cb: Timeout'], output: ['1. Script Start', '2. asyncFn Start', '3. Script End', '4. asyncFn Resume'], activeZone: 'stack', note: "✓ '4. asyncFn Resume' logged to console." },
  // 18
  { codeLine: 5, stack: [], heapSaved: null, microQueue: [], macroQueue: ['cb: Timeout'], output: ['1. Script Start', '2. asyncFn Start', '3. Script End', '4. asyncFn Resume'], activeZone: 'loop', note: 'asyncFn finishes & resolves its outer promise. Call Stack is empty again.' },
  // 19
  { codeLine: 0, stack: [], heapSaved: null, microQueue: [], macroQueue: ['cb: Timeout'], output: ['1. Script Start', '2. asyncFn Start', '3. Script End', '4. asyncFn Resume'], activeZone: 'macro', note: 'Event loop: Microtasks are 0. Now picks ONE macrotask: cb (setTimeout).' },
  // 20
  { codeLine: 8, stack: ['cb: Timeout()'], heapSaved: null, microQueue: [], macroQueue: [], output: ['1. Script Start', '2. asyncFn Start', '3. Script End', '4. asyncFn Resume'], activeZone: 'stack', note: 'Timeout callback pushed to Call Stack.' },
  // 21
  { codeLine: 8, stack: [], heapSaved: null, microQueue: [], macroQueue: [], output: ['1. Script Start', '2. asyncFn Start', '3. Script End', '4. asyncFn Resume', '5. Timeout'], activeZone: 'idle', note: "✓ '5. Timeout' logged! All queues and stacks empty." },
  // 22
  { codeLine: 0, stack: [], heapSaved: null, microQueue: [], macroQueue: [], output: ['1. Script Start', '2. asyncFn Start', '3. Script End', '4. asyncFn Resume', '5. Timeout'], activeZone: 'idle', note: '💡 FINAL LESSON: await null beat setTimeout(0) because await resumes via the VIP Microtask Queue!' },
]

const cur = computed(() => states[Math.min(s.value, states.length - 1)])

const codeLines = [
  { num: 1, text: "async function asyncFn() {" },
  { num: 2, text: "  console.log('2. asyncFn Start');" },
  { num: 3, text: "  await null; // Suspend & yield" },
  { num: 4, text: "  console.log('4. asyncFn Resume');" },
  { num: 5, text: "}" },
  { num: 6, text: "" },
  { num: 7, text: "console.log('1. Script Start');" },
  { num: 8, text: "setTimeout(() => console.log('5. Timeout'), 0);" },
  { num: 9, text: "asyncFn();" },
  { num: 10, text: "console.log('3. Script End');" },
]
</script>

<template>
  <div class="h-full flex flex-col gap-1.5 select-none text-slate-800">
    <!-- Header -->
    <div class="flex items-center gap-3 pb-1 border-b-2 border-slate-200">
      <span class="text-2xl">⚡</span>
      <div>
        <h2 class="text-xl font-black text-slate-900 leading-tight">async / await Live Execution Trace</h2>
        <p class="text-xs text-slate-500">Chapter 6 of 6 · Interactive Simulator</p>
      </div>
      <div class="ml-auto px-2 py-1 rounded-lg bg-sky-600 text-white text-xs font-bold">Step {{ s }}/22</div>
    </div>

    <!-- Active Note Banner -->
    <div
      class="rounded-xl px-3 py-1.5 text-xs font-bold border-2 flex items-center gap-2 transition-all duration-300"
      :class="{
        'bg-sky-50 border-sky-400 text-sky-900': cur.activeZone === 'stack',
        'bg-purple-50 border-purple-400 text-purple-900': cur.activeZone === 'heap',
        'bg-violet-50 border-violet-400 text-violet-900': cur.activeZone === 'micro',
        'bg-amber-50 border-amber-400 text-amber-900': cur.activeZone === 'macro',
        'bg-emerald-50 border-emerald-400 text-emerald-900': cur.activeZone === 'loop',
        'bg-slate-50 border-slate-300 text-slate-700': cur.activeZone === 'idle' || cur.activeZone === 'code',
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
            <span class="text-violet-400 text-[9px] font-mono">async/await + Timer</span>
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
          <span class="text-amber-400 font-bold">Mental Model:</span> <code class="text-sky-300">await X</code> pauses and returns a Promise, placing its continuation into the microtask queue.
        </div>
      </div>

      <!-- Col 2: Call Stack & Heap (3 cols) -->
      <div class="col-span-3 flex flex-col gap-2">
        <!-- Call Stack -->
        <div
          class="border-2 rounded-xl p-2 flex-1 flex flex-col transition-all duration-300"
          :class="cur.activeZone === 'stack' ? 'border-sky-500 bg-sky-50/50 shadow-md ring-2 ring-sky-200' : 'border-slate-200 bg-white'"
        >
          <div class="text-[10px] font-black text-slate-700 uppercase tracking-wider mb-1 flex items-center justify-between">
            <span>⚡ Call Stack</span>
            <span class="bg-sky-100 text-sky-800 px-1 rounded text-[8px] font-bold">{{ cur.stack.length }} frames</span>
          </div>

          <div class="flex-1 flex flex-col-reverse gap-1 justify-start">
            <div v-if="cur.stack.length === 0" class="flex items-center justify-center h-16 border-2 border-dashed border-slate-200 rounded-lg">
              <span class="text-[9px] text-slate-400 italic">Stack empty</span>
            </div>
            <div
              v-for="(f, i) in cur.stack" :key="f + i"
              class="rounded px-2 py-1 text-[9px] font-bold border transition-all duration-300 shadow-sm"
              :class="i === cur.stack.length - 1 ? 'bg-sky-600 text-white border-sky-700 animate-pulse' : 'bg-sky-50 border-sky-300 text-sky-900'"
            >
              <div class="flex items-center justify-between">
                <span class="truncate">{{ f }}</span>
                <span class="text-[7px] px-1 rounded bg-black/20 shrink-0 ml-1">{{ i === cur.stack.length - 1 ? 'TOP' : '' }}</span>
              </div>
            </div>
          </div>
        </div>

        <!-- Heap Suspended Context -->
        <div
          class="border-2 rounded-xl p-2 transition-all duration-300"
          :class="cur.activeZone === 'heap' ? 'border-purple-500 bg-purple-50 ring-2 ring-purple-200 shadow-md' : 'border-slate-200 bg-white'"
        >
          <div class="text-[9px] font-black text-purple-900 uppercase tracking-wider mb-0.5 flex items-center justify-between">
            <span>📦 Heap Suspended Context</span>
            <span v-if="cur.heapSaved" class="bg-purple-200 text-purple-900 px-1 rounded text-[8px] font-bold">PAUSED</span>
          </div>
          <div v-if="!cur.heapSaved" class="text-[8px] text-slate-400 italic">No paused coroutines</div>
          <div v-else class="text-[9px] font-mono text-purple-900 font-bold bg-purple-100 p-1 rounded border border-purple-300 truncate">
            {{ cur.heapSaved }}
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
              class="rounded-md px-2 py-1 text-[9px] font-bold bg-violet-600 text-white flex items-center justify-between shadow-sm animate-pulse"
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
              class="rounded-md px-2 py-1 text-[9px] font-bold bg-amber-400 text-amber-950 border border-amber-500 flex items-center justify-between shadow-sm"
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
          <span class="text-emerald-400 text-[9px] font-mono">{{ cur.output.length }}/5</span>
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
          await null beats setTimeout(0)!
        </div>
      </div>
    </div>
  </div>
</template>
