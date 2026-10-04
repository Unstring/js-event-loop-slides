<script setup lang="ts">
import { computed } from 'vue'

const props = defineProps<{ step?: number }>()
const s = computed(() => Math.min(Math.max(props.step ?? 0, 0), 22))

const microtasks = [
  { id: 2, name: 'Promise.then / catch / finally', spec: 'ECMAScript (Jobs)', icon: '🤝', priority: 'VIP' },
  { id: 3, name: 'queueMicrotask(() => {})', spec: 'HTML5 Standard', icon: '⚡', priority: 'VIP' },
  { id: 4, name: 'MutationObserver', spec: 'DOM Level 4', icon: '👁️', priority: 'VIP' },
  { id: 5, name: 'process.nextTick (Node.js)', spec: 'libuv internal (super-VIP)', icon: '🚀', priority: 'P0' },
]

const macrotasks = [
  { id: 7, name: 'setTimeout / setInterval', spec: 'HTML5 Timers', icon: '⏱️', priority: 'Normal' },
  { id: 8, name: 'DOM Events (click, keypress)', spec: 'UI Events Spec', icon: '🖱️', priority: 'Normal' },
  { id: 9, name: 'Network Fetch & Ajax callbacks', spec: 'XHR / Fetch Spec', icon: '📡', priority: 'Normal' },
  { id: 10, name: 'setImmediate / I/O (Node.js)', spec: 'libuv Check Phase', icon: '⚙️', priority: 'Normal' },
]

const comparisonRows = [
  { id: 12, label: 'Execution Rule', micro: 'DRAIN ALL to 0 (Complete exhaustion)', macro: 'Exactly ONE task per turn', highlight: true },
  { id: 13, label: 'Spec Authority', micro: 'ECMAScript Language Spec (Jobs)', macro: 'HTML5 / WHATWG Event Loop Spec', highlight: false },
  { id: 14, label: 'Yield to Render?', micro: 'NEVER yields until empty (blocks UI)', macro: 'YES, yields every turn for 60fps', highlight: true },
  { id: 15, label: 'Self-recursion', micro: 'Freezes tab! (Microtask starvation)', macro: 'Safe! Browser paints between calls', highlight: true },
]
</script>

<template>
  <div class="h-full flex flex-col gap-2 select-none text-slate-800">
    <!-- Header -->
    <div class="flex items-center gap-3 pb-1 border-b-2 border-slate-200">
      <span class="text-2xl">⚖️</span>
      <div>
        <h2 class="text-xl font-black text-slate-900 leading-tight">Microtasks vs Macrotasks</h2>
        <p class="text-xs text-slate-500">Chapter 5 of 6 · Architectural Mechanics</p>
      </div>
      <div class="ml-auto px-2 py-1 rounded-lg bg-sky-600 text-white text-xs font-bold">Step {{ s }}/22</div>
    </div>

    <!-- Subtitle banner -->
    <div class="bg-slate-50 border border-slate-200 rounded-lg px-3 py-1 text-xs text-slate-700 flex items-center justify-between">
      <span class="font-medium">
        <strong class="text-violet-700">Microtasks (VIP Jobs)</strong> run immediately after current synchronous frame, while
        <strong class="text-amber-700">Macrotasks (Tasks)</strong> wait for their turn one-by-one.
      </span>
      <span class="text-[10px] text-slate-500 font-mono">HTML5 § 8.1.6</span>
    </div>

    <!-- Main Grid -->
    <div class="grid grid-cols-2 gap-3 flex-1 min-h-0">
      <!-- LEFT: Two Queues Breakdown -->
      <div class="flex flex-col gap-2">
        <!-- Microtask Box -->
        <div
          class="border-2 rounded-xl p-2.5 bg-violet-50/60 border-violet-300 transition-all duration-300 flex flex-col gap-1.5"
          :class="s >= 1 ? 'opacity-100 translate-y-0' : 'opacity-0 translate-y-2'"
        >
          <div class="flex items-center justify-between">
            <span class="text-xs font-black text-violet-900 flex items-center gap-1.5">
              <span>👑</span> Microtasks Queue (VIP Priority)
            </span>
            <span class="bg-violet-600 text-white font-bold text-[9px] px-2 py-0.5 rounded-full">Drain 100%</span>
          </div>

          <div class="grid grid-cols-1 gap-1">
            <div
              v-for="item in microtasks" :key="item.id"
              class="rounded-lg p-1.5 border text-xs font-medium flex items-center justify-between transition-all duration-300"
              :class="[
                s >= item.id ? 'opacity-100 scale-100 bg-white border-violet-200 shadow-sm' : 'opacity-0 scale-95 bg-white/40 border-transparent',
              ]"
            >
              <div class="flex items-center gap-1.5 truncate">
                <span>{{ item.icon }}</span>
                <span class="font-bold text-violet-950 font-mono text-[11px]">{{ item.name }}</span>
              </div>
              <span class="text-[9px] text-violet-600 shrink-0 bg-violet-100 px-1 rounded font-semibold">{{ item.spec }}</span>
            </div>
          </div>
        </div>

        <!-- Macrotask Box -->
        <div
          class="border-2 rounded-xl p-2.5 bg-amber-50/60 border-amber-300 transition-all duration-300 flex flex-col gap-1.5"
          :class="s >= 6 ? 'opacity-100 translate-y-0' : 'opacity-0 translate-y-2'"
        >
          <div class="flex items-center justify-between">
            <span class="text-xs font-black text-amber-900 flex items-center gap-1.5">
              <span>⏱️</span> Macrotasks Queue (Standard Tasks)
            </span>
            <span class="bg-amber-600 text-white font-bold text-[9px] px-2 py-0.5 rounded-full">1 Per Turn</span>
          </div>

          <div class="grid grid-cols-1 gap-1">
            <div
              v-for="item in macrotasks" :key="item.id"
              class="rounded-lg p-1.5 border text-xs font-medium flex items-center justify-between transition-all duration-300"
              :class="[
                s >= item.id ? 'opacity-100 scale-100 bg-white border-amber-200 shadow-sm' : 'opacity-0 scale-95 bg-white/40 border-transparent',
              ]"
            >
              <div class="flex items-center gap-1.5 truncate">
                <span>{{ item.icon }}</span>
                <span class="font-bold text-amber-950 font-mono text-[11px]">{{ item.name }}</span>
              </div>
              <span class="text-[9px] text-amber-700 shrink-0 bg-amber-100 px-1 rounded font-semibold">{{ item.spec }}</span>
            </div>
          </div>
        </div>
      </div>

      <!-- RIGHT: The Rules & Starvation Code -->
      <div class="flex flex-col gap-2">
        <!-- Comparison Table -->
        <div
          class="border-2 border-slate-200 rounded-xl p-2.5 bg-white flex flex-col transition-all duration-300"
          :class="s >= 11 ? 'opacity-100' : 'opacity-0'"
        >
          <div class="text-[11px] font-black text-slate-700 uppercase tracking-wider mb-1.5">
            📊 The Concurrency Contract
          </div>

          <div class="flex flex-col gap-1">
            <div
              v-for="row in comparisonRows" :key="row.id"
              class="border rounded-lg p-1.5 text-[11px] flex flex-col transition-all duration-300"
              :class="[
                s >= row.id ? 'opacity-100 translate-x-0' : 'opacity-0 translate-x-2',
                row.highlight ? 'bg-slate-50 border-slate-300' : 'bg-white border-slate-100'
              ]"
            >
              <div class="font-black text-slate-800 text-[10px] mb-0.5">{{ row.label }}</div>
              <div class="grid grid-cols-2 gap-2 text-[10px]">
                <div class="text-violet-900 bg-violet-50 p-1 rounded border border-violet-200">
                  <span class="font-bold">Micro:</span> {{ row.micro }}
                </div>
                <div class="text-amber-900 bg-amber-50 p-1 rounded border border-amber-200">
                  <span class="font-bold">Macro:</span> {{ row.macro }}
                </div>
              </div>
            </div>
          </div>
        </div>

        <!-- Danger Zone: Starvation Example -->
        <div
          class="border-2 border-rose-300 rounded-xl p-2.5 bg-rose-50/70 transition-all duration-300 flex-1 flex flex-col justify-between"
          :class="s >= 16 ? 'opacity-100 scale-100' : 'opacity-0 scale-95'"
        >
          <div>
            <div class="text-[11px] font-black text-rose-900 flex items-center justify-between mb-1">
              <span class="flex items-center gap-1">⚠️ Danger: Microtask Starvation</span>
              <span class="bg-rose-200 text-rose-900 text-[9px] px-1.5 py-0.2 rounded font-mono font-bold">UI Freeze</span>
            </div>
            <div class="text-[10px] text-rose-800 leading-snug mb-1.5">
              Because microtasks drain to exhaustion, a self-scheduling microtask will <strong class="underline">NEVER</strong> let the event loop render or handle clicks!
            </div>

            <!-- Code snippet -->
            <div class="bg-slate-900 rounded-lg p-2 font-mono text-[10px] text-slate-200">
              <div class="text-rose-400">// This will PERMANENTLY freeze the browser tab:</div>
              <div>function starve() {</div>
              <div class="pl-3 text-amber-300">queueMicrotask(starve); // infinitely queues</div>
              <div>}</div>
              <div>starve(); <span class="text-slate-500">// Stack empties, but microtasks NEVER empty!</span></div>
            </div>
          </div>

          <div class="mt-1 text-[10px] text-slate-700 bg-white/80 p-1.5 rounded border border-rose-200">
            <strong>Rule of Thumb:</strong> Use microtasks for immediate state updates & Promise chaining; use macrotasks (<code class="bg-slate-100 px-1">setTimeout(fn, 0)</code>) when you need to <strong>yield to the browser UI</strong>!
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
