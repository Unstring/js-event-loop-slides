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
  { id: 12, label: 'Execution Rule', micro: 'DRAIN ALL to 0 (Complete exhaustion)', macro: 'Exactly ONE task per turn' },
  { id: 13, label: 'Yield to Render?', micro: 'NEVER yields until 100% empty (blocks UI)', macro: 'YES, yields every turn for 60fps' },
  { id: 14, label: 'Infinite Recursion', micro: 'Freezes tab! (Microtask starvation)', macro: 'Safe! Browser renders between turns' },
]
</script>

<template>
  <div class="h-full flex flex-col justify-between select-none text-slate-800 text-xs">
    <!-- Header -->
    <div class="flex items-center gap-2 pb-1 border-b border-slate-200 shrink-0">
      <span class="text-xl">⚖️</span>
      <div>
        <h2 class="text-base font-black text-slate-900 leading-tight">Microtasks vs Macrotasks: Priority Architecture</h2>
        <p class="text-[10px] text-slate-500">Chapter 5 of 6 · Two-Tier Concurrency & Starvation Hazards</p>
      </div>
      <div class="ml-auto flex items-center gap-1.5">
        <span class="px-2 py-0.5 rounded bg-violet-100 text-violet-900 text-[10px] font-bold">Two-Tier Queue System</span>
        <div class="px-2 py-0.5 rounded bg-sky-600 text-white text-[10px] font-bold">Step {{ s }}/22</div>
      </div>
    </div>

    <!-- 2 Columns -->
    <div class="grid grid-cols-2 gap-2 flex-1 min-h-0 py-1">
      <!-- LEFT: Two Queues Breakdown (Steps 1-10) -->
      <div class="flex flex-col gap-1.5 justify-between min-h-0">
        <!-- Microtask Box -->
        <div
          class="border-2 rounded-lg p-2 bg-violet-50/50 border-violet-300 flex flex-col gap-1 transition-all duration-300"
          :class="s >= 1 ? 'opacity-100' : 'opacity-25'"
        >
          <div class="flex items-center justify-between text-[10px]">
            <span class="font-black text-violet-900 flex items-center gap-1">
              <span>👑</span> Microtasks (VIP Priority Queue)
            </span>
            <span class="bg-violet-600 text-white font-bold text-[8px] px-1.5 py-0.2 rounded">Drain 100%</span>
          </div>

          <div class="grid grid-cols-1 gap-0.5">
            <div
              v-for="item in microtasks" :key="item.id"
              class="rounded p-1 border text-[9px] flex items-center justify-between transition-all duration-200"
              :class="s >= item.id ? 'opacity-100 bg-white border-violet-200 shadow-xs' : 'opacity-20 border-transparent'"
            >
              <div class="flex items-center gap-1 truncate">
                <span>{{ item.icon }}</span>
                <span class="font-bold text-violet-950 font-mono text-[9px]">{{ item.name }}</span>
              </div>
              <span class="text-[8px] text-violet-700 bg-violet-100 px-1 rounded font-semibold">{{ item.spec }}</span>
            </div>
          </div>
        </div>

        <!-- Macrotask Box -->
        <div
          class="border-2 rounded-lg p-2 bg-amber-50/50 border-amber-300 flex flex-col gap-1 transition-all duration-300"
          :class="s >= 6 ? 'opacity-100' : 'opacity-25'"
        >
          <div class="flex items-center justify-between text-[10px]">
            <span class="font-black text-amber-900 flex items-center gap-1">
              <span>⏱️</span> Macrotasks (Standard Task Queue)
            </span>
            <span class="bg-amber-600 text-white font-bold text-[8px] px-1.5 py-0.2 rounded">1 Per Turn</span>
          </div>

          <div class="grid grid-cols-1 gap-0.5">
            <div
              v-for="item in macrotasks" :key="item.id"
              class="rounded p-1 border text-[9px] flex items-center justify-between transition-all duration-200"
              :class="s >= item.id ? 'opacity-100 bg-white border-amber-200 shadow-xs' : 'opacity-20 border-transparent'"
            >
              <div class="flex items-center gap-1 truncate">
                <span>{{ item.icon }}</span>
                <span class="font-bold text-amber-950 font-mono text-[9px]">{{ item.name }}</span>
              </div>
              <span class="text-[8px] text-amber-800 bg-amber-100 px-1 rounded font-semibold">{{ item.spec }}</span>
            </div>
          </div>
        </div>
      </div>

      <!-- RIGHT: The Concurrency Rules & Starvation Code (Steps 11-22) -->
      <div class="flex flex-col gap-1.5 justify-between min-h-0">
        <!-- Concurrency Table (Steps 11-14) -->
        <div
          class="border border-slate-200 rounded-lg p-2 bg-white flex flex-col gap-1 transition-all duration-300"
          :class="s >= 11 ? 'opacity-100' : 'opacity-25'"
        >
          <div class="text-[10px] font-black text-slate-700 uppercase tracking-wider flex justify-between">
            <span>📊 Concurrency Rules Comparison</span>
            <span class="text-[8px] text-slate-400 font-mono">Steps 11-14</span>
          </div>

          <div class="flex flex-col gap-0.5">
            <div
              v-for="row in comparisonRows" :key="row.id"
              class="border rounded p-1 text-[9px] flex flex-col transition-all duration-200"
              :class="s >= row.id ? 'bg-slate-50 border-slate-300' : 'border-transparent text-slate-300'"
            >
              <div class="font-bold text-slate-800 text-[8px]">{{ row.label }}</div>
              <div class="grid grid-cols-2 gap-1 text-[8px] mt-0.5">
                <div class="text-violet-900 bg-violet-50 px-1 py-0.5 rounded border border-violet-200 truncate">
                  <strong>Micro:</strong> {{ row.micro }}
                </div>
                <div class="text-amber-900 bg-amber-50 px-1 py-0.5 rounded border border-amber-200 truncate">
                  <strong>Macro:</strong> {{ row.macro }}
                </div>
              </div>
            </div>
          </div>
        </div>

        <!-- Danger Zone: Starvation Example (Steps 15-22) -->
        <div
          class="border border-rose-300 rounded-lg p-2 bg-rose-50/60 flex flex-col justify-between transition-all duration-300 shrink-0"
          :class="s >= 15 ? 'opacity-100' : 'opacity-25'"
        >
          <div>
            <div class="text-[10px] font-black text-rose-900 flex items-center justify-between mb-0.5">
              <span>⚠️ Danger: Microtask Starvation</span>
              <span class="bg-rose-200 text-rose-900 text-[8px] px-1 rounded font-mono font-bold">UI Freeze</span>
            </div>
            <div class="text-[9px] text-rose-800 leading-snug mb-1">
              Because microtasks drain to 0, recursive microtasks permanently block rendering and user clicks!
            </div>

            <!-- Code snippet -->
            <div class="bg-slate-900 rounded p-1.5 font-mono text-[9px] text-slate-200 leading-tight">
              <div class="text-rose-400">// This will PERMANENTLY freeze the browser tab:</div>
              <div>function starve() { queueMicrotask(starve); }</div>
              <div>starve(); <span class="text-slate-500">// Stack empties, but microtasks NEVER empty!</span></div>
            </div>
          </div>

          <div class="text-[8px] text-slate-700 bg-white/90 p-1 rounded border border-rose-200 mt-1">
            <strong>Rule:</strong> Use microtasks for Promise reactions; use <code class="bg-slate-100 px-1 font-bold">setTimeout(fn, 0)</code> when you must <strong>yield to UI render</strong>!
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
