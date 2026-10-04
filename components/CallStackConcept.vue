<script setup lang="ts">
import { computed } from 'vue'

const props = defineProps<{ step?: number }>()
const s = computed(() => Math.min(Math.max(props.step ?? 0, 0), 22))

const lifoPlates = [
  { id: 1, label: 'Plate 1: main()', sub: 'Bottom (Pushed First, Popped Last)', color: 'bg-slate-100 border-slate-300 text-slate-800' },
  { id: 2, label: 'Plate 2: greet()', sub: 'Middle Frame', color: 'bg-sky-100 border-sky-300 text-sky-900' },
  { id: 3, label: 'Plate 3: getName()', sub: 'Sub-routine Frame', color: 'bg-violet-100 border-violet-300 text-violet-900' },
  { id: 4, label: 'Plate 4: getTitle()', sub: 'TOP Frame ← Currently Running CPU', color: 'bg-emerald-600 border-emerald-700 text-white font-bold' },
]

const ecAttributes = [
  { id: 7, label: 'Variable Environment', val: 'Stores let, const, var declarations & values' },
  { id: 8, label: 'Lexical Scope Chain', val: 'Pointers to parent scopes for closures & variables' },
  { id: 9, label: 'ThisBinding', val: 'Bound to calling object, undefined, or globalThis' },
]

const codeLines = [
  { n: 1, text: "function getTitle() {", activeOn: [14] },
  { n: 2, text: "  return 'Dr.';", activeOn: [15] },
  { n: 3, text: "}", activeOn: [] },
  { n: 4, text: "function getName(name) {", activeOn: [13] },
  { n: 5, text: "  return getTitle() + ' ' + name;", activeOn: [14, 16] },
  { n: 6, text: "}", activeOn: [] },
  { n: 7, text: "function greet(name) {", activeOn: [11] },
  { n: 8, text: "  const msg = getName(name);", activeOn: [12, 17] },
  { n: 9, text: "  console.log(msg);", activeOn: [18] },
  { n: 10, text: "}", activeOn: [] },
  { n: 11, text: "greet('Alice');", activeOn: [10] },
]

const isLineActive = (activeOn: number[]) => {
  return activeOn.includes(s.value)
}

const overflowRules = [
  { id: 19, code: 'function recurse() { recurse(); }', note: 'Infinite call recursion without base condition' },
  { id: 20, code: 'recurse(); // RangeError', note: 'Maximum call stack size exceeded (~10,000 frames)' },
  { id: 21, code: 'Solution 1: Base Case', note: 'Ensure termination condition (if (n <= 1) return 1;)' },
  { id: 22, code: 'Solution 2: Macrotask / Loop', note: 'Iterate with loop or trampoline to keep stack O(1)' },
]
</script>

<template>
  <div class="h-full flex flex-col justify-between select-none text-slate-800 text-xs">
    <!-- Header -->
    <div class="flex items-center gap-2 pb-1 border-b border-slate-200 shrink-0">
      <span class="text-xl">📚</span>
      <div>
        <h2 class="text-base font-black text-slate-900 leading-tight">The Call Stack & Execution Context</h2>
        <p class="text-[10px] text-slate-500">Chapter 2 of 5 · LIFO Mechanics & Stack Anatomy</p>
      </div>
      <div class="ml-auto flex items-center gap-1.5">
        <span class="px-2 py-0.5 rounded bg-blue-100 text-blue-900 text-[10px] font-bold">LIFO Data Structure</span>
        <div class="px-2 py-0.5 rounded bg-sky-600 text-white text-[10px] font-bold">Step {{ s }}/22</div>
      </div>
    </div>

    <!-- 2 Columns -->
    <div class="grid grid-cols-2 gap-2 flex-1 min-h-0 py-1">
      <!-- LEFT: Concept, LIFO, and Overflow -->
      <div class="flex flex-col gap-1.5 justify-between min-h-0">
        <!-- LIFO Stack Analogy (Steps 1-6) -->
        <div class="border border-slate-200 rounded-lg p-2 bg-slate-50/70 flex flex-col gap-1">
          <div class="text-[10px] font-black text-slate-700 uppercase tracking-wider flex justify-between">
            <span>🥞 LIFO: Last-In, First-Out Principle</span>
            <span class="text-[9px] text-slate-400 font-mono">Steps 1-6</span>
          </div>
          <div class="flex flex-col-reverse gap-0.5">
            <div
              v-for="p in lifoPlates" :key="p.id"
              class="border rounded px-2 py-0.5 text-[10px] flex items-center justify-between transition-all duration-200"
              :class="[
                p.color,
                s >= p.id ? 'opacity-100 scale-100' : 'opacity-20 scale-98'
              ]"
            >
              <span>{{ p.label }}</span>
              <span class="text-[8px] opacity-80">{{ p.sub }}</span>
            </div>
          </div>
        </div>

        <!-- Execution Context Anatomy (Steps 7-9) -->
        <div
          class="border border-violet-200 rounded-lg p-2 bg-violet-50/40 flex flex-col gap-1 transition-all duration-300"
          :class="s >= 7 ? 'opacity-100' : 'opacity-25'"
        >
          <div class="text-[10px] font-black text-violet-800 uppercase tracking-wider flex justify-between">
            <span>📦 Anatomy of an Execution Context</span>
            <span class="text-[9px] text-violet-600 font-mono">Steps 7-9</span>
          </div>
          <div class="flex flex-col gap-0.5">
            <div
              v-for="item in ecAttributes" :key="item.id"
              class="px-1.5 py-0.5 rounded border text-[9px] flex items-center justify-between transition-all duration-200"
              :class="s >= item.id ? 'bg-white border-violet-300 text-violet-950 font-medium' : 'bg-transparent border-transparent text-slate-400'"
            >
              <span class="font-bold text-violet-900">{{ item.label }}:</span>
              <span class="text-slate-600">{{ item.val }}</span>
            </div>
          </div>
        </div>

        <!-- Stack Overflow Guard (Steps 19-22) -->
        <div
          class="border border-rose-200 rounded-lg p-1.5 bg-rose-50/50 flex flex-col gap-0.5 transition-all duration-300 shrink-0"
          :class="s >= 19 ? 'opacity-100' : 'opacity-25'"
        >
          <div class="text-[10px] font-black text-rose-800 uppercase tracking-wider flex justify-between">
            <span>💥 Stack Overflow & Prevention</span>
            <span class="text-[9px] text-rose-600 font-mono">Steps 19-22</span>
          </div>
          <div class="grid grid-cols-2 gap-1 text-[9px]">
            <div
              v-for="o in overflowRules" :key="o.id"
              class="p-1 rounded border transition-all duration-200"
              :class="s >= o.id ? 'bg-white border-rose-300 text-rose-950' : 'border-transparent text-slate-400'"
            >
              <div class="font-mono font-bold truncate text-[8px]">{{ o.code }}</div>
              <div class="text-[8px] text-slate-500 leading-tight">{{ o.note }}</div>
            </div>
          </div>
        </div>
      </div>

      <!-- RIGHT: Code Tracing & Execution Pointer -->
      <div class="flex flex-col gap-1.5 justify-between min-h-0">
        <!-- Code snippet with exact line indicator -->
        <div class="border border-slate-300 rounded-lg p-2 bg-slate-900 text-slate-100 flex flex-col justify-between flex-1 min-h-0">
          <div>
            <div class="text-[10px] font-black text-slate-400 uppercase tracking-wider mb-1 flex justify-between">
              <span>🔍 Trace Nested Calls</span>
              <span class="text-[9px] text-amber-400 font-mono">Steps 10-18</span>
            </div>
            <div class="font-mono text-[10px] flex flex-col gap-0.5">
              <div
                v-for="line in codeLines" :key="line.n"
                class="px-1.5 py-0.2 rounded flex items-center justify-between transition-all duration-150"
                :class="{
                  'bg-amber-500/30 text-amber-200 font-bold border-l-2 border-amber-400': isLineActive(line.activeOn),
                  'text-slate-400': !isLineActive(line.activeOn)
                }"
              >
                <div class="flex items-center gap-1.5">
                  <span class="text-[8px] text-slate-600 select-none w-3 text-right">{{ line.n }}</span>
                  <span>{{ line.text }}</span>
                </div>
                <span v-if="isLineActive(line.activeOn)" class="text-[8px] text-amber-300 font-sans font-bold animate-pulse">
                  ▶ RUNNING
                </span>
              </div>
            </div>
          </div>

          <!-- Execution state tracker -->
          <div class="bg-slate-800 p-1.5 rounded border border-slate-700 text-[10px] font-mono mt-1">
            <div class="text-slate-400 text-[8px]">CURRENT STACK FRAME:</div>
            <div class="text-emerald-300 font-bold">
              <span v-if="s < 10">Idle (Awaiting step 10)</span>
              <span v-else-if="s === 10">main() -> greet('Alice')</span>
              <span v-else-if="s === 11">greet('Alice') begins</span>
              <span v-else-if="s === 12">greet calls getName('Alice')</span>
              <span v-else-if="s === 13">getName calls getTitle()</span>
              <span v-else-if="s === 14">getTitle() pushed to TOP</span>
              <span v-else-if="s === 15">getTitle returns 'Dr.' → POPS</span>
              <span v-else-if="s === 16">getName returns 'Dr. Alice' → POPS</span>
              <span v-else-if="s === 17">greet receives 'Dr. Alice'</span>
              <span v-else-if="s === 18">console.log printed → POPS → Stack EMPTY ✓</span>
              <span v-else>Stack is Empty (Event Loop Checkpoint)</span>
            </div>
          </div>
        </div>

        <!-- 4 Golden Stack Laws -->
        <div class="border border-sky-300 rounded-lg p-1.5 bg-sky-50 text-sky-950 shrink-0">
          <div class="text-[9px] font-black text-sky-900 mb-0.5">📋 The 4 Immutable Stack Rules:</div>
          <div class="grid grid-cols-2 gap-x-2 text-[8px] text-sky-900 leading-tight">
            <div>1. Call pushes frame; return pops frame</div>
            <div>2. Only TOP frame has CPU execution</div>
            <div>3. Synchronous execution is 100% blocking</div>
            <div>4. Event Loop ONLY runs when Stack is EMPTY</div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
