<script setup lang="ts">
import { computed } from 'vue'

const props = defineProps<{ step?: number }>()
const s = computed(() => Math.min(Math.max(props.step ?? 0, 0), 22))

interface Frame {
  name: string
  line: string
  color: string
  textColor: string
  isTop?: boolean
}

interface StepState {
  frames: Frame[]
  currentLine: number
  output: string[]
  note: string
  phase: string
  radarTarget?: 'stack' | 'micro' | 'render' | 'macro' | 'blocked'
}

const states: StepState[] = [
  // 0 (initial)
  { frames: [], currentLine: 0, output: [], note: 'Program not started. Stack is empty.', phase: 'idle' },
  // 1
  { frames: [{ name: 'main()', line: 'L1', color: 'bg-slate-200 border-slate-300', textColor: 'text-slate-800' }], currentLine: 1, output: [], note: 'Global execution starts. main() pushed onto Call Stack.', phase: 'push' },
  // 2 - Fixed: Line 12 (not line 11) is greet('Alice')!
  { frames: [{ name: 'main()', line: 'L1', color: 'bg-slate-200 border-slate-300', textColor: 'text-slate-800' }, { name: "greet('Alice')", line: 'L12', color: 'bg-sky-500 border-sky-600', textColor: 'text-white', isTop: true }], currentLine: 12, output: [], note: "Line 12: greet('Alice') called. Frame pushed to stack.", phase: 'push' },
  // 3
  { frames: [{ name: 'main()', line: 'L1', color: 'bg-slate-200 border-slate-300', textColor: 'text-slate-800' }, { name: "greet('Alice')", line: 'L8', color: 'bg-sky-500 border-sky-600', textColor: 'text-white' }, { name: "getName('Alice')", line: 'L8', color: 'bg-violet-500 border-violet-600', textColor: 'text-white', isTop: true }], currentLine: 8, output: [], note: "Line 8: getName('Alice') called from greet. New frame on top.", phase: 'push' },
  // 4
  { frames: [{ name: 'main()', line: 'L1', color: 'bg-slate-200 border-slate-300', textColor: 'text-slate-800' }, { name: "greet('Alice')", line: 'L8', color: 'bg-sky-500 border-sky-600', textColor: 'text-white' }, { name: "getName('Alice')", line: 'L5', color: 'bg-violet-500 border-violet-600', textColor: 'text-white' }, { name: 'getTitle()', line: 'L5', color: 'bg-emerald-600 border-emerald-700', textColor: 'text-white', isTop: true }], currentLine: 5, output: [], note: "Line 5: getTitle() called from getName. Stack depth: 4.", phase: 'push' },
  // 5
  { frames: [{ name: 'main()', line: 'L1', color: 'bg-slate-200 border-slate-300', textColor: 'text-slate-800' }, { name: "greet('Alice')", line: 'L8', color: 'bg-sky-500 border-sky-600', textColor: 'text-white' }, { name: "getName('Alice')", line: 'L5', color: 'bg-violet-500 border-violet-600', textColor: 'text-white' }, { name: 'getTitle()', line: 'L2', color: 'bg-emerald-600 border-emerald-700', textColor: 'text-white', isTop: true }], currentLine: 2, output: [], note: "Line 2: getTitle executes return 'Dr.' — preparing to pop.", phase: 'exec' },
  // 6 - pop getTitle
  { frames: [{ name: 'main()', line: 'L1', color: 'bg-slate-200 border-slate-300', textColor: 'text-slate-800' }, { name: "greet('Alice')", line: 'L8', color: 'bg-sky-500 border-sky-600', textColor: 'text-white' }, { name: "getName('Alice')", line: 'L5', color: 'bg-violet-500 border-violet-600', textColor: 'text-white', isTop: true }], currentLine: 5, output: [], note: "getTitle() returns 'Dr.' → frame POPPED. getName resumes.", phase: 'pop' },
  // 7 - pop getName
  { frames: [{ name: 'main()', line: 'L1', color: 'bg-slate-200 border-slate-300', textColor: 'text-slate-800' }, { name: "greet('Alice')", line: 'L8', color: 'bg-sky-500 border-sky-600', textColor: 'text-white', isTop: true }], currentLine: 8, output: [], note: "getName returns 'Dr. Alice' → frame POPPED. greet resumes.", phase: 'pop' },
  // 8
  { frames: [{ name: 'main()', line: 'L1', color: 'bg-slate-200 border-slate-300', textColor: 'text-slate-800' }, { name: "greet('Alice')", line: 'L9', color: 'bg-sky-500 border-sky-600', textColor: 'text-white', isTop: true }], currentLine: 9, output: [], note: "Line 9: greet has msg = 'Dr. Alice'. Calls console.log(msg).", phase: 'exec' },
  // 9
  { frames: [{ name: 'main()', line: 'L1', color: 'bg-slate-200 border-slate-300', textColor: 'text-slate-800' }, { name: "greet('Alice')", line: 'L9', color: 'bg-sky-500 border-sky-600', textColor: 'text-white' }, { name: "console.log('Dr. Alice')", line: 'L9', color: 'bg-amber-500 border-amber-600', textColor: 'text-white', isTop: true }], currentLine: 9, output: [], note: "console.log pushed to stack. Executes synchronously.", phase: 'push' },
  // 10 - log executes
  { frames: [{ name: 'main()', line: 'L1', color: 'bg-slate-200 border-slate-300', textColor: 'text-slate-800' }, { name: "greet('Alice')", line: 'L9', color: 'bg-sky-500 border-sky-600', textColor: 'text-white', isTop: true }], currentLine: 9, output: ['Dr. Alice'], note: "✓ 'Dr. Alice' printed. console.log frame popped.", phase: 'pop' },
  // 11 - pop greet
  { frames: [{ name: 'main()', line: 'L1', color: 'bg-slate-200 border-slate-300', textColor: 'text-slate-800', isTop: true }], currentLine: 12, output: ['Dr. Alice'], note: "greet() returns → frame POPPED. Only main() remains.", phase: 'pop' },
  // 12 - pop main
  { frames: [], currentLine: 0, output: ['Dr. Alice'], note: "🔑 main() finishes! Call Stack is completely EMPTY! Event Loop awakens!", phase: 'empty', radarTarget: 'stack' },
  // 13 - Event Loop Scanner ACTIVE
  { frames: [], currentLine: 0, output: ['Dr. Alice'], note: "📡 EVENT LOOP SENSOR: Step 1 — Confirms Call Stack has 0 frames.", phase: 'eventloop', radarTarget: 'stack' },
  // 14
  { frames: [], currentLine: 0, output: ['Dr. Alice'], note: "📡 EVENT LOOP SENSOR: Step 2 — Checks VIP Microtask Queue (Promises).", phase: 'eventloop', radarTarget: 'micro' },
  // 15
  { frames: [], currentLine: 0, output: ['Dr. Alice'], note: "📡 EVENT LOOP SENSOR: Step 3 — Checks 60 FPS Render Pipeline (requestAnimationFrame).", phase: 'eventloop', radarTarget: 'render' },
  // 16
  { frames: [], currentLine: 0, output: ['Dr. Alice'], note: "📡 EVENT LOOP SENSOR: Step 4 — Checks Macrotask Queue (setTimeout, I/O, clicks).", phase: 'eventloop', radarTarget: 'macro' },
  // 17
  { frames: [], currentLine: 0, output: ['Dr. Alice'], note: "🛡️ RUN-TO-COMPLETION RULE: Event Loop can NEVER interrupt a running function!", phase: 'eventloop', radarTarget: 'blocked' },
  // 18
  { frames: [], currentLine: 0, output: ['Dr. Alice'], note: "⚠️ If a function runs a long synchronous loop, the Event Loop is locked out.", phase: 'eventloop', radarTarget: 'blocked' },
  // 19
  { frames: [], currentLine: 0, output: ['Dr. Alice'], note: "💡 Cooperative Scheduling: To avoid UI freeze, slice big tasks with setTimeout(fn, 0).", phase: 'eventloop', radarTarget: 'macro' },
  // 20
  { frames: [], currentLine: 0, output: ['Dr. Alice'], note: "✅ All frames popped cleanly in exact LIFO reverse order.", phase: 'eventloop', radarTarget: 'stack' },
  // 21
  { frames: [], currentLine: 0, output: ['Dr. Alice'], note: "🏆 Memory reclaimed: Execution context variables garbage collected.", phase: 'eventloop', radarTarget: 'stack' },
  // 22
  { frames: [], currentLine: 0, output: ['Dr. Alice'], note: "Next up: Diving deep into the Event Loop Heartbeat and Host Environments!", phase: 'eventloop', radarTarget: 'stack' },
]

const codeLines2 = [
  "function getTitle() {",
  "  return 'Dr.'",
  "}",
  "function getName(name) {",
  "  return getTitle() + name",
  "}",
  "function greet(name) {",
  "  const msg = getName(name)",
  "  console.log(msg)",
  "}",
  "",
  "greet('Alice')",
]

const cur = computed(() => states[Math.min(s.value, states.length - 1)])
</script>

<template>
  <div class="h-full flex flex-col justify-between select-none text-slate-800 text-xs">
    <!-- Header -->
    <div class="flex items-center gap-2 pb-1 border-b border-slate-200 shrink-0">
      <span class="text-xl">⚡</span>
      <div>
        <h2 class="text-base font-black text-slate-900 leading-tight">Call Stack — Live Execution & Event Loop Handshake</h2>
        <p class="text-[10px] text-slate-500">Chapter 2 of 5 · Interactive Execution Trace</p>
      </div>
      <div class="ml-auto flex items-center gap-1.5">
        <span class="px-2 py-0.5 rounded text-[10px] font-black border uppercase"
          :class="s >= 12 ? 'bg-amber-100 border-amber-300 text-amber-900' : 'bg-sky-100 border-sky-300 text-sky-800'"
        >
          {{ s >= 12 ? 'Event Loop Active' : 'Stack Executing' }}
        </span>
        <div class="px-2 py-0.5 rounded bg-sky-600 text-white text-[10px] font-bold">Step {{ s }}/22</div>
      </div>
    </div>

    <!-- Active Banner -->
    <div
      class="border rounded-lg px-2.5 py-1 text-[11px] font-medium flex items-center gap-1.5 transition-all duration-300 shrink-0"
      :class="s >= 12 ? 'bg-amber-50 border-amber-300 text-amber-900' : 'bg-sky-50 border-sky-300 text-sky-900'"
    >
      <span class="animate-pulse font-bold">▶</span>
      <span>{{ cur.note }}</span>
    </div>

    <!-- Main Grid -->
    <div class="grid grid-cols-12 gap-2 flex-1 min-h-0 py-1">
      <!-- Col 1: Code Panel (4 cols) -->
      <div class="col-span-4 border border-slate-300 rounded-lg p-2 bg-slate-900 text-slate-100 flex flex-col justify-between">
        <div>
          <div class="text-[10px] font-black text-slate-400 uppercase tracking-wider mb-1 flex justify-between">
            <span>📝 Source Code</span>
            <span class="text-[9px] text-amber-400 font-mono">12 Lines</span>
          </div>
          <div class="font-mono text-[10px] flex flex-col gap-0.5">
            <div
              v-for="(line, i) in codeLines2" :key="i"
              class="px-1.5 py-0.2 rounded flex items-center gap-1.5 transition-all duration-150"
              :class="{
                'bg-amber-500/30 text-amber-200 font-bold border-l-2 border-amber-400': cur.currentLine === i + 1,
                'text-slate-500': cur.currentLine !== i + 1 && line,
                'opacity-0': !line
              }"
            >
              <span class="text-[8px] text-slate-600 select-none w-3 text-right">{{ i + 1 }}</span>
              <span class="truncate">{{ line }}</span>
            </div>
          </div>
        </div>

        <!-- Terminal Output -->
        <div class="bg-slate-800 p-1.5 rounded border border-slate-700 text-[10px]">
          <div class="text-[8px] text-slate-400 uppercase tracking-wider mb-0.5">Console Output:</div>
          <div v-if="cur.output.length === 0" class="text-slate-500 text-[9px] italic">awaiting output...</div>
          <div v-for="(o, i) in cur.output" :key="i" class="font-mono text-emerald-400 font-bold text-[10px]">
            ▶ "{{ o }}"
          </div>
        </div>
      </div>

      <!-- Col 2: Call Stack (4 cols) -->
      <div class="col-span-4 border-2 border-slate-200 rounded-lg p-2 bg-slate-50 flex flex-col justify-between">
        <div class="flex items-center justify-between text-[10px] font-black text-slate-700 uppercase">
          <span>📚 Call Stack</span>
          <span class="bg-sky-100 text-sky-800 px-1 rounded text-[9px] font-mono">{{ cur.frames.length }} frames</span>
        </div>

        <!-- Stack Container -->
        <div class="flex-1 flex flex-col-reverse gap-1 justify-start py-1 min-h-0">
          <div v-if="cur.frames.length === 0" class="flex-1 flex items-center justify-center border-2 border-dashed border-slate-300 rounded-lg">
            <div class="text-center p-2">
              <span class="text-2xl block" :class="s >= 12 ? 'animate-bounce text-emerald-600' : 'text-slate-400'">✓</span>
              <span class="text-[10px] font-bold" :class="s >= 12 ? 'text-emerald-700' : 'text-slate-400'">
                {{ s >= 12 ? 'Call Stack is EMPTY' : 'Idle' }}
              </span>
            </div>
          </div>
          <div
            v-for="(frame, i) in cur.frames" :key="frame.name + i"
            class="rounded px-2 py-1 text-[10px] font-bold border flex items-center justify-between transition-all duration-200 shadow-xs"
            :class="[frame.color, frame.textColor, frame.isTop ? 'ring-2 ring-amber-400 ring-offset-1 scale-[1.01]' : '']"
          >
            <span class="font-mono truncate">{{ frame.name }}</span>
            <div class="flex items-center gap-1 shrink-0 ml-1">
              <span class="text-[7px] px-1 rounded bg-black/20 font-mono">{{ frame.line }}</span>
              <span v-if="frame.isTop" class="text-[7px] px-1 rounded bg-amber-500 text-white font-black">TOP</span>
            </div>
          </div>
        </div>

        <div class="text-[9px] text-slate-500 text-center bg-white border border-slate-200 rounded py-0.5">
          Push on function call · Pop on return (LIFO)
        </div>
      </div>

      <!-- Col 3: Event Loop Handshake Radar (4 cols) -->
      <div
        class="col-span-4 border-2 rounded-lg p-2 flex flex-col justify-between transition-all duration-300"
        :class="s >= 12 ? 'border-amber-400 bg-amber-50/40 shadow-sm' : 'border-slate-200 bg-slate-50 opacity-50'"
      >
        <div>
          <div class="flex items-center justify-between text-[10px] font-black uppercase mb-1">
            <span class="flex items-center gap-1 text-amber-900">
              <span :class="s >= 12 ? 'animate-spin' : ''">↻</span> Event Loop Handshake
            </span>
            <span class="text-[8px] bg-amber-200 text-amber-900 px-1 rounded font-bold">Steps 13-22</span>
          </div>
          <div class="text-[9px] text-slate-600 mb-2 leading-tight">
            The Event Loop monitors the Call Stack. When depth hits 0, it triggers its 4 inspection checkpoints:
          </div>

          <!-- 4 Checkpoints visualizer -->
          <div class="flex flex-col gap-1">
            <div
              class="border rounded p-1 text-[9px] flex items-center justify-between transition-all duration-300"
              :class="cur.radarTarget === 'stack' ? 'bg-emerald-100 border-emerald-400 text-emerald-950 font-bold scale-[1.02] ring-1 ring-emerald-300' : 'bg-white border-slate-200 text-slate-600'"
            >
              <span>1. Stack Check: Is Stack Empty?</span>
              <span class="text-[8px] px-1 rounded font-mono font-bold" :class="cur.frames.length === 0 ? 'bg-emerald-600 text-white' : 'bg-slate-200'">
                {{ cur.frames.length === 0 ? 'YES (0)' : 'BUSY' }}
              </span>
            </div>

            <div
              class="border rounded p-1 text-[9px] flex items-center justify-between transition-all duration-300"
              :class="cur.radarTarget === 'micro' ? 'bg-violet-100 border-violet-400 text-violet-950 font-bold scale-[1.02] ring-1 ring-violet-300' : 'bg-white border-slate-200 text-slate-600'"
            >
              <span>2. Microtasks Check (VIP Queue)</span>
              <span class="text-[8px] px-1 rounded font-mono font-bold" :class="cur.radarTarget === 'micro' ? 'bg-violet-600 text-white animate-pulse' : 'bg-slate-200'">
                0 Pending
              </span>
            </div>

            <div
              class="border rounded p-1 text-[9px] flex items-center justify-between transition-all duration-300"
              :class="cur.radarTarget === 'render' ? 'bg-rose-100 border-rose-400 text-rose-950 font-bold scale-[1.02] ring-1 ring-rose-300' : 'bg-white border-slate-200 text-slate-600'"
            >
              <span>3. 60 FPS Render Pipeline</span>
              <span class="text-[8px] px-1 rounded font-mono font-bold" :class="cur.radarTarget === 'render' ? 'bg-rose-600 text-white animate-pulse' : 'bg-slate-200'">
                V-Sync Pass
              </span>
            </div>

            <div
              class="border rounded p-1 text-[9px] flex items-center justify-between transition-all duration-300"
              :class="cur.radarTarget === 'macro' ? 'bg-amber-100 border-amber-400 text-amber-950 font-bold scale-[1.02] ring-1 ring-amber-300' : 'bg-white border-slate-200 text-slate-600'"
            >
              <span>4. Macrotask Queue (Timers/IO)</span>
              <span class="text-[8px] px-1 rounded font-mono font-bold" :class="cur.radarTarget === 'macro' ? 'bg-amber-600 text-white animate-pulse' : 'bg-slate-200'">
                1 Per Turn
              </span>
            </div>
          </div>
        </div>

        <!-- Run to completion takeaway -->
        <div class="border border-sky-300 rounded p-1.5 bg-white text-[9px] text-sky-950 mt-1">
          <span class="font-bold text-sky-900">Key Law:</span> JavaScript code executes to completion. The Event Loop <strong>cannot</strong> steal CPU while a stack frame is active!
        </div>
      </div>
    </div>
  </div>
</template>
