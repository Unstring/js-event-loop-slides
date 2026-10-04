<script setup lang="ts">
import { computed } from 'vue'
const props = defineProps<{ step?: number }>()
const s = computed(() => Math.min(Math.max(props.step ?? 0, 0), 22))

const lifoItems = [
  { id: 1, label: '🍽️  Plate 1 (first in)', sub: 'bottom of stack', color: 'bg-sky-100 border-sky-300' },
  { id: 2, label: '🍽️  Plate 2', sub: 'middle', color: 'bg-sky-200 border-sky-400' },
  { id: 3, label: '🍽️  Plate 3 (last in, first out)', sub: 'TOP ← removed first', color: 'bg-sky-600 border-sky-700 text-white' },
]

const ecParts = [
  { id: 7, label: 'Variable Environment', detail: 'var declarations hoisted to undefined, let/const in TDZ', color: 'bg-violet-50 border-violet-300 text-violet-900' },
  { id: 9, label: 'Scope Chain', detail: 'Reference to outer environment for closure variable lookup', color: 'bg-amber-50 border-amber-300 text-amber-900' },
  { id: 11, label: 'this binding', detail: 'Global object / undefined (strict) / the calling object', color: 'bg-emerald-50 border-emerald-300 text-emerald-900' },
]

const codeLines = [
  { n: 1, text: "function getTitle() {", step: 0, indent: 0 },
  { n: 2, text: "  return 'Dr.'", step: 14, indent: 1, active: true },
  { n: 3, text: "}", step: 0, indent: 0 },
  { n: 4, text: "function getName(name) {", step: 0, indent: 0 },
  { n: 5, text: "  return getTitle() + name", step: 13, indent: 1, active: true },
  { n: 6, text: "}", step: 0, indent: 0 },
  { n: 7, text: "function greet(name) {", step: 0, indent: 0 },
  { n: 8, text: "  const msg = getName(name)", step: 12, indent: 1, active: true },
  { n: 9, text: "  console.log(msg)", step: 17, indent: 1, active: true },
  { n: 10, text: "}", step: 0, indent: 0 },
  { n: 11, text: "greet('Alice')", step: 11, active: true },
]

const overflow = [
  { id: 19, text: 'function recurse() { recurse() }', tag: 'Infinite recursion' },
  { id: 20, text: 'recurse() // RangeError: Maximum call stack size exceeded', tag: 'Stack overflow ~10,000 frames' },
  { id: 21, text: '// Fix: add base case', tag: 'if (n === 0) return 1' },
  { id: 22, text: '// Or: convert to iteration / use trampoline', tag: 'Cooperative, O(1) stack' },
]
</script>

<template>
  <div class="h-full flex flex-col gap-2 select-none">
    <div class="flex items-center gap-3 pb-1 border-b-2 border-slate-200">
      <span class="text-2xl">📚</span>
      <div>
        <h2 class="text-xl font-black text-slate-900 leading-tight">The Call Stack</h2>
        <p class="text-xs text-slate-500">Chapter 2 of 5 · Concept & Code</p>
      </div>
      <div class="ml-auto px-2 py-1 rounded-lg bg-sky-600 text-white text-xs font-bold">Step {{ s }}/22</div>
    </div>

    <div class="grid grid-cols-2 gap-3 flex-1 min-h-0">
      <!-- LEFT -->
      <div class="flex flex-col gap-2 min-h-0">
        <!-- LIFO Analogy -->
        <div>
          <div class="text-xs font-black text-slate-600 uppercase tracking-wider mb-1">📌 Stack = LIFO (Last In, First Out)</div>
          <div class="flex flex-col-reverse gap-1">
            <div
              v-for="item in lifoItems" :key="item.id"
              class="border-2 rounded-lg px-3 py-1 text-xs font-bold flex justify-between items-center transition-all duration-400"
              :class="[item.color, s >= item.id ? 'opacity-100' : 'opacity-0 -translate-y-2']"
              style="transition: opacity 0.3s, transform 0.3s"
            >
              <span>{{ item.label }}</span>
              <span class="text-[10px] font-normal opacity-75">{{ item.sub }}</span>
            </div>
          </div>
        </div>

        <!-- Execution Context Anatomy -->
        <div :class="s >= 7 ? 'opacity-100' : 'opacity-0'" style="transition: opacity 0.4s">
          <div class="text-xs font-black text-violet-700 uppercase tracking-wider mb-1">🏗️ Each Frame = Execution Context</div>
          <div class="flex flex-col gap-1">
            <div
              v-for="part in ecParts" :key="part.id"
              class="border-2 rounded-lg px-2 py-1 text-xs transition-all duration-300"
              :class="[part.color, s >= part.id ? 'opacity-100' : 'opacity-0']"
              style="transition: opacity 0.35s"
            >
              <span class="font-black">{{ part.label }}:</span>
              <span class="font-medium ml-1">{{ part.detail }}</span>
            </div>
          </div>
        </div>

        <!-- Stack overflow -->
        <div :class="s >= 19 ? 'opacity-100' : 'opacity-0'" style="transition: opacity 0.4s">
          <div class="text-xs font-black text-red-700 uppercase tracking-wider mb-1">💥 Stack Overflow</div>
          <div class="bg-slate-900 rounded-lg p-2 font-mono text-[10px] leading-relaxed">
            <div
              v-for="line in overflow" :key="line.id"
              class="mb-0.5 transition-all duration-300"
              :class="s >= line.id ? 'opacity-100' : 'opacity-0'"
            >
              <span class="text-amber-300">{{ line.text }}</span>
              <span class="text-emerald-400 ml-2">// {{ line.tag }}</span>
            </div>
          </div>
        </div>
      </div>

      <!-- RIGHT: Code trace -->
      <div class="flex flex-col gap-2">
        <div class="text-xs font-black text-slate-600 uppercase tracking-wider mb-1">🔍 Code Trace — Step by Step</div>
        <div class="bg-slate-900 rounded-xl p-3 font-mono text-[11px] leading-relaxed">
          <div
            v-for="line in codeLines" :key="line.n"
            class="flex items-start gap-2 rounded px-1 mb-0.5 transition-all duration-200"
            :class="line.active && s >= (line.step ?? 0) ? 'bg-amber-500/20' : ''"
          >
            <span class="text-slate-600 w-4 shrink-0 text-right text-[9px] mt-0.5">{{ line.n }}</span>
            <span
              :class="line.active && s >= (line.step ?? 0) ? 'text-amber-300 font-black' : 'text-slate-300'"
              :style="{ paddingLeft: (line.indent ?? 0) * 12 + 'px' }"
            >{{ line.text }}</span>
            <span v-if="line.active && s >= (line.step ?? 0)" class="ml-auto text-[8px] text-amber-400 animate-pulse shrink-0">▶ executing</span>
          </div>
        </div>

        <!-- Current execution pointer note -->
        <div
          class="bg-amber-50 border-2 border-amber-300 rounded-xl px-3 py-2 transition-all duration-400 text-xs"
          :class="s >= 11 ? 'opacity-100' : 'opacity-0'"
        >
          <div class="font-black text-amber-800 mb-1">🔑 Currently Executing:</div>
          <div class="text-amber-900 font-mono font-medium">
            <span v-if="s < 12">greet('Alice') → line 7</span>
            <span v-else-if="s < 13">greet → calls getName → line 8</span>
            <span v-else-if="s < 14">getName → calls getTitle → line 5</span>
            <span v-else-if="s < 15">getTitle → returns 'Dr.' → line 2</span>
            <span v-else-if="s < 16">getName → returns 'Dr. Alice' → pops</span>
            <span v-else-if="s < 17">greet → has msg = 'Dr. Alice' → line 9</span>
            <span v-else-if="s < 18">console.log('Dr. Alice') → pops</span>
            <span v-else>greet → pops → Stack EMPTY ✓</span>
          </div>
        </div>

        <!-- Key rule callout -->
        <div
          class="mt-auto bg-sky-50 border-2 border-sky-400 rounded-xl px-3 py-2 transition-all duration-500"
          :class="s >= 18 ? 'opacity-100' : 'opacity-0'"
        >
          <div class="text-xs font-black text-sky-900">📋 Rules to Remember</div>
          <div class="text-[11px] text-sky-900 mt-1 space-y-0.5">
            <div>① Every function call pushes a <strong>frame</strong> onto the stack</div>
            <div>② The <strong>top frame</strong> has exclusive control of the CPU thread</div>
            <div>③ Return pops the frame — stack can only empty from the top</div>
            <div>④ An empty stack is the <strong>only time</strong> the Event Loop can act</div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
