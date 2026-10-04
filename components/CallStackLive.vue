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
}

const states: StepState[] = [
  // 0 (initial)
  { frames: [], currentLine: 0, output: [], note: 'Program not started. Stack is empty.', phase: 'idle' },
  // 1
  { frames: [{ name: 'main()', line: 'L1', color: 'bg-slate-200', textColor: 'text-slate-800' }], currentLine: 1, output: [], note: 'Global execution starts. main() pushes onto stack.', phase: 'push' },
  // 2
  { frames: [{ name: 'main()', line: 'L11', color: 'bg-slate-200', textColor: 'text-slate-800' }, { name: "greet('Alice')", line: 'L11', color: 'bg-sky-400', textColor: 'text-white', isTop: true }], currentLine: 11, output: [], note: "Line 11: greet('Alice') is called. New frame pushed.", phase: 'push' },
  // 3
  { frames: [{ name: 'main()', line: 'L11', color: 'bg-slate-200', textColor: 'text-slate-800' }, { name: "greet('Alice')", line: 'L8', color: 'bg-sky-400', textColor: 'text-white' }, { name: "getName('Alice')", line: 'L8', color: 'bg-violet-400', textColor: 'text-white', isTop: true }], currentLine: 8, output: [], note: "Line 8: getName(name) called from greet. New frame on top.", phase: 'push' },
  // 4
  { frames: [{ name: 'main()', line: 'L11', color: 'bg-slate-200', textColor: 'text-slate-800' }, { name: "greet('Alice')", line: 'L8', color: 'bg-sky-400', textColor: 'text-white' }, { name: "getName('Alice')", line: 'L5', color: 'bg-violet-400', textColor: 'text-white' }, { name: 'getTitle()', line: 'L5', color: 'bg-emerald-400', textColor: 'text-white', isTop: true }], currentLine: 5, output: [], note: "Line 5: getTitle() called from getName. Stack depth: 4.", phase: 'push' },
  // 5
  { frames: [{ name: 'main()', line: 'L11', color: 'bg-slate-200', textColor: 'text-slate-800' }, { name: "greet('Alice')", line: 'L8', color: 'bg-sky-400', textColor: 'text-white' }, { name: "getName('Alice')", line: 'L5', color: 'bg-violet-400', textColor: 'text-white' }, { name: 'getTitle()', line: 'L2', color: 'bg-emerald-400', textColor: 'text-white', isTop: true }], currentLine: 2, output: [], note: "Inside getTitle: return 'Dr.' — prepares to pop.", phase: 'exec' },
  // 6 - pop getTitle
  { frames: [{ name: 'main()', line: 'L11', color: 'bg-slate-200', textColor: 'text-slate-800' }, { name: "greet('Alice')", line: 'L8', color: 'bg-sky-400', textColor: 'text-white' }, { name: "getName('Alice')", line: 'L5', color: 'bg-violet-400', textColor: 'text-white', isTop: true }], currentLine: 5, output: [], note: "getTitle() returned 'Dr.' → frame POPPED. Control returns to getName.", phase: 'pop' },
  // 7 - pop getName
  { frames: [{ name: 'main()', line: 'L11', color: 'bg-slate-200', textColor: 'text-slate-800' }, { name: "greet('Alice')", line: 'L8', color: 'bg-sky-400', textColor: 'text-white', isTop: true }], currentLine: 8, output: [], note: "getName returned 'Dr. Alice' → frame POPPED. Back in greet.", phase: 'pop' },
  // 8
  { frames: [{ name: 'main()', line: 'L11', color: 'bg-slate-200', textColor: 'text-slate-800' }, { name: "greet('Alice')", line: 'L9', color: 'bg-sky-400', textColor: 'text-white', isTop: true }], currentLine: 9, output: [], note: "greet: msg = 'Dr. Alice'. Now calls console.log(msg).", phase: 'exec' },
  // 9
  { frames: [{ name: 'main()', line: 'L11', color: 'bg-slate-200', textColor: 'text-slate-800' }, { name: "greet('Alice')", line: 'L9', color: 'bg-sky-400', textColor: 'text-white' }, { name: "console.log('Dr. Alice')", line: 'L9', color: 'bg-amber-400', textColor: 'text-white', isTop: true }], currentLine: 9, output: [], note: "console.log pushed. Native API call — executes synchronously.", phase: 'push' },
  // 10 - log executes
  { frames: [{ name: 'main()', line: 'L11', color: 'bg-slate-200', textColor: 'text-slate-800' }, { name: "greet('Alice')", line: 'L9', color: 'bg-sky-400', textColor: 'text-white', isTop: true }], currentLine: 9, output: ['Dr. Alice'], note: "'Dr. Alice' printed. console.log popped. Back in greet.", phase: 'pop' },
  // 11 - pop greet
  { frames: [{ name: 'main()', line: 'L11', color: 'bg-slate-200', textColor: 'text-slate-800', isTop: true }], currentLine: 11, output: ['Dr. Alice'], note: "greet() returns → frame POPPED. Only main() remains.", phase: 'pop' },
  // 12 - pop main
  { frames: [], currentLine: 12, output: ['Dr. Alice'], note: "main() completes → Stack is now COMPLETELY EMPTY. ← This is the critical moment!", phase: 'empty' },
  // 13 onward - quiz steps
  { frames: [], currentLine: 0, output: ['Dr. Alice'], note: "❓ Q: What happens when the stack empties? → The Event Loop wakes up!", phase: 'question' },
  { frames: [], currentLine: 0, output: ['Dr. Alice'], note: "✅ Event Loop checks: any microtasks? any macrotasks?", phase: 'answer' },
  { frames: [], currentLine: 0, output: ['Dr. Alice'], note: "Stack was empty → Event Loop can now safely process queued callbacks.", phase: 'answer' },
  { frames: [], currentLine: 0, output: ['Dr. Alice'], note: "This is why async code (setTimeout, fetch, Promises) never interrupts running code.", phase: 'answer' },
  { frames: [], currentLine: 0, output: ['Dr. Alice'], note: "Key rule: JS code runs to COMPLETION — no preemption, no interruption.", phase: 'answer' },
  { frames: [], currentLine: 0, output: ['Dr. Alice'], note: "Run-to-completion guarantee: once a function starts, it finishes before anything else.", phase: 'summary' },
  { frames: [], currentLine: 0, output: ['Dr. Alice'], note: "This makes reasoning about JS code much simpler — no race conditions inside JS!", phase: 'summary' },
  { frames: [], currentLine: 0, output: ['Dr. Alice'], note: "Summary: push on call → execute top → pop on return → repeat until empty.", phase: 'summary' },
  { frames: [], currentLine: 0, output: ['Dr. Alice'], note: "🏆 You now understand the Call Stack. Next: the Event Loop!", phase: 'done' },
]

const codeText = `function getTitle() {
  return 'Dr.'
}
function getName(name) {
  return getTitle() + name
}
function greet(name) {
  const msg = getName(name)
  console.log(msg)
}

greet('Alice')`

const codeLines2 = codeText.split('\n')

const cur = computed(() => states[Math.min(s.value, states.length - 1)])

const phaseColor = computed(() => ({
  idle: 'bg-slate-100 border-slate-300 text-slate-600',
  push: 'bg-sky-100 border-sky-400 text-sky-800',
  exec: 'bg-amber-100 border-amber-400 text-amber-800',
  pop: 'bg-rose-100 border-rose-400 text-rose-800',
  empty: 'bg-emerald-100 border-emerald-500 text-emerald-900',
  question: 'bg-violet-100 border-violet-400 text-violet-900',
  answer: 'bg-emerald-100 border-emerald-400 text-emerald-900',
  summary: 'bg-sky-100 border-sky-400 text-sky-900',
  done: 'bg-amber-100 border-amber-400 text-amber-900',
}[cur.value.phase] || 'bg-slate-100 border-slate-300'))
</script>

<template>
  <div class="h-full flex flex-col gap-2 select-none">
    <div class="flex items-center gap-3 pb-1 border-b-2 border-slate-200">
      <span class="text-2xl">⚡</span>
      <div>
        <h2 class="text-xl font-black text-slate-900 leading-tight">Call Stack — Live Execution Trace</h2>
        <p class="text-xs text-slate-500">Chapter 2 of 5 · Interactive</p>
      </div>
      <div class="ml-auto flex items-center gap-2">
        <span class="px-2 py-0.5 rounded text-xs font-black border-2 uppercase transition-all duration-300" :class="phaseColor">{{ cur.phase }}</span>
        <div class="px-2 py-1 rounded-lg bg-sky-600 text-white text-xs font-bold">Step {{ s }}/22</div>
      </div>
    </div>

    <div class="grid grid-cols-5 gap-3 flex-1 min-h-0">
      <!-- Code panel (2 cols) -->
      <div class="col-span-2 flex flex-col gap-2">
        <div class="text-xs font-black text-slate-600 uppercase tracking-wider">📝 Source Code</div>
        <div class="bg-slate-900 rounded-xl p-3 font-mono text-[11px] leading-relaxed flex-1">
          <div
            v-for="(line, i) in codeLines2" :key="i"
            class="flex gap-2 rounded px-1 transition-all duration-200"
            :class="cur.currentLine === i + 1 ? 'bg-amber-500/30' : ''"
          >
            <span class="text-slate-600 w-4 text-right shrink-0 text-[9px] mt-0.5">{{ i + 1 }}</span>
            <span :class="cur.currentLine === i + 1 ? 'text-amber-200 font-black' : 'text-slate-300'">{{ line || ' ' }}</span>
            <span v-if="cur.currentLine === i + 1" class="ml-auto text-amber-400 text-[8px] animate-pulse shrink-0">◀</span>
          </div>
        </div>
        <!-- Output console -->
        <div class="bg-slate-800 rounded-lg px-3 py-2">
          <div class="text-[10px] text-slate-500 font-bold uppercase mb-1">Console Output</div>
          <div v-if="cur.output.length === 0" class="text-slate-600 text-[10px] italic">awaiting output...</div>
          <div v-for="(o, i) in cur.output" :key="i" class="font-mono text-emerald-400 font-bold text-xs">▶ {{ o }}</div>
        </div>
      </div>

      <!-- Stack visualizer (2 cols) -->
      <div class="col-span-2 flex flex-col gap-2">
        <div class="text-xs font-black text-slate-600 uppercase tracking-wider flex items-center justify-between">
          <span>📚 Call Stack</span>
          <span class="text-slate-400 font-normal text-[10px]">↑ grows up</span>
        </div>
        <div class="flex-1 border-2 border-slate-200 rounded-xl bg-slate-50 flex flex-col-reverse p-2 gap-1 relative">
          <div
            v-if="cur.frames.length === 0"
            class="absolute inset-0 flex items-center justify-center"
          >
            <div class="text-center">
              <div class="text-3xl mb-1" :class="s >= 12 ? 'animate-bounce' : ''">✓</div>
              <div class="text-xs font-black" :class="s >= 12 ? 'text-emerald-600' : 'text-slate-400'">
                {{ s >= 12 ? 'EMPTY — Event Loop wakes!' : 'Empty' }}
              </div>
            </div>
          </div>
          <div
            v-for="(frame, i) in cur.frames" :key="frame.name + i"
            class="rounded-lg px-2 py-1.5 text-xs font-bold flex items-center justify-between border-2 transition-all duration-300 shadow-sm"
            :class="[frame.color, frame.textColor, frame.isTop ? 'ring-2 ring-offset-1 ring-amber-400 scale-[1.02]' : 'opacity-85']"
          >
            <span class="font-mono truncate">{{ frame.name }}</span>
            <div class="flex items-center gap-1 shrink-0 ml-1">
              <span class="text-[8px] font-bold opacity-80 bg-black/10 px-1 rounded">{{ frame.line }}</span>
              <span v-if="frame.isTop" class="text-[8px] font-black bg-amber-500 text-white px-1 rounded">TOP</span>
            </div>
          </div>
        </div>
        <div class="text-center text-[10px] font-bold text-slate-500 bg-slate-200 rounded py-0.5">
          Stack Depth: {{ cur.frames.length }}
        </div>
      </div>

      <!-- Notes panel (1 col) -->
      <div class="col-span-1 flex flex-col gap-2">
        <div class="text-xs font-black text-slate-600 uppercase tracking-wider">💬 What's Happening</div>
        <div
          class="border-2 rounded-xl p-2 text-xs font-medium leading-snug transition-all duration-400 flex-1"
          :class="phaseColor"
        >
          {{ cur.note }}
        </div>

        <!-- Phase legend -->
        <div class="space-y-1 text-[10px]">
          <div class="flex items-center gap-1"><span class="w-2 h-2 rounded-full bg-sky-400 shrink-0"></span><span class="font-bold text-sky-700">PUSH</span><span class="text-slate-500"> = call</span></div>
          <div class="flex items-center gap-1"><span class="w-2 h-2 rounded-full bg-amber-400 shrink-0"></span><span class="font-bold text-amber-700">EXEC</span><span class="text-slate-500"> = run</span></div>
          <div class="flex items-center gap-1"><span class="w-2 h-2 rounded-full bg-rose-400 shrink-0"></span><span class="font-bold text-rose-700">POP</span><span class="text-slate-500"> = return</span></div>
          <div class="flex items-center gap-1"><span class="w-2 h-2 rounded-full bg-emerald-500 shrink-0"></span><span class="font-bold text-emerald-700">EMPTY</span><span class="text-slate-500"> = event loop</span></div>
        </div>
      </div>
    </div>
  </div>
</template>
