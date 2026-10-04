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

// Steps 15-22 Starvation State Simulation
const starvationQueue = computed(() => {
  if (s.value === 16) return ['starve_#1']
  if (s.value === 17) return ['starve_#2']
  if (s.value === 18) return ['starve_#3', 'starve_#4']
  if (s.value >= 19 && s.value <= 20) return ['starve_#999...', 'starve_#1000...']
  if (s.value >= 21) return []
  return []
})

const starvationNote = computed(() => {
  if (s.value === 15) return 'Step 15: The Starvation Function declared. Recursive self-scheduling microtask.'
  if (s.value === 16) return 'Step 16: Initial call: starve() pushes starve_#1 into Microtask VIP Queue.'
  if (s.value === 17) return 'Step 17: starve_#1 executes → queues starve_#2 before stack finishes.'
  if (s.value === 18) return 'Step 18: Queue NEVER drains to 0! Event Loop is permanently trapped in Phase 2.'
  if (s.value === 19) return 'Step 19: Macrotasks (setTimeout, clicks) are STARVED — cannot run!'
  if (s.value === 20) return 'Step 20: 60 FPS Render pipeline is BLOCKED — tab is 100% frozen!'
  if (s.value === 21) return 'Step 21 [The Fix]: Use setTimeout(chunk, 0) to yield to the renderer between turns!'
  if (s.value >= 22) return 'Step 22 [Rule of Thumb]: Microtasks for data integrity; Macrotasks for UI fairness.'
  return 'Two-Tier Concurrency Architecture: Microtasks drain to exhaustion; Macrotasks run 1 per turn.'
})
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
        <span class="px-2 py-0.5 rounded text-[10px] font-bold border"
          :class="s >= 15 ? 'bg-rose-100 border-rose-300 text-rose-900' : 'bg-violet-100 border-violet-300 text-violet-900'"
        >
          {{ s >= 15 ? (s >= 21 ? 'Yield Solution' : 'Starvation Hazard') : 'Two-Tier Queue System' }}
        </span>
        <div class="px-2 py-0.5 rounded bg-sky-600 text-white text-[10px] font-bold">Step {{ s }}/22</div>
      </div>
    </div>

    <!-- Active Banner -->
    <div
      class="border rounded-lg px-2.5 py-1 text-[11px] font-medium flex items-center gap-1.5 transition-all duration-300 shrink-0"
      :class="s >= 15 ? (s >= 21 ? 'bg-emerald-50 border-emerald-300 text-emerald-950 font-bold' : 'bg-rose-50 border-rose-300 text-rose-900 font-bold') : 'bg-slate-50 border-slate-200 text-slate-700'"
    >
      <span class="animate-pulse">▶</span>
      <span>{{ starvationNote }}</span>
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

      <!-- RIGHT: Concurrency Rules & Dynamic Starvation Simulator (Steps 11-22) -->
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

        <!-- Danger Zone: Interactive Starvation Simulation (Steps 15-22) -->
        <div
          class="border-2 rounded-lg p-2 flex flex-col justify-between transition-all duration-300 shrink-0"
          :class="{
            'bg-rose-50/70 border-rose-300 opacity-100': s >= 15 && s < 21,
            'bg-emerald-50/70 border-emerald-400 opacity-100': s >= 21,
            'bg-slate-50 border-slate-200 opacity-25': s < 15
          }"
        >
          <div>
            <div class="text-[10px] font-black flex items-center justify-between mb-1">
              <span class="flex items-center gap-1" :class="s >= 21 ? 'text-emerald-900' : 'text-rose-900'">
                <span>{{ s >= 21 ? '✅ Solution: Cooperative Yielding' : '⚠️ Danger: Microtask Starvation' }}</span>
              </span>
              <span class="text-[8px] px-1 rounded font-mono font-bold"
                :class="s >= 21 ? 'bg-emerald-200 text-emerald-900' : 'bg-rose-200 text-rose-900'"
              >
                {{ s >= 21 ? '60 FPS RESTORED' : (s >= 19 ? 'UI COMPLETELY FROZEN' : 'Active Loop') }}
              </span>
            </div>

            <!-- Code snippet: changes dynamically between Step 15-20 and Step 21-22 -->
            <div class="bg-slate-900 rounded p-1.5 font-mono text-[8.5px] text-slate-200 leading-tight">
              <div v-if="s < 21">
                <div class="text-rose-400">// Starvation Bug (Infinite Microtask recursion):</div>
                <div :class="s >= 16 ? 'text-amber-300 font-bold' : ''">function starve() {</div>
                <div class="pl-2" :class="s >= 17 ? 'text-rose-300 font-bold' : ''">queueMicrotask(starve); // queues next microtask!</div>
                <div>}</div>
                <div :class="s >= 16 ? 'text-emerald-400 font-bold' : ''">starve();</div>
              </div>
              <div v-else>
                <div class="text-emerald-400">// Cooperative Batching (Yields to Browser Render):</div>
                <div class="text-emerald-300 font-bold">function chunkWork() {</div>
                <div class="pl-2 text-sky-300">doPartialComputation();</div>
                <div class="pl-2 text-amber-300 font-bold">setTimeout(chunkWork, 0); // Yields to UI render & clicks!</div>
                <div class="text-emerald-300">}</div>
              </div>
            </div>

            <!-- Real-time Queue & UI Status Gauge (Steps 16-22) -->
            <div class="mt-1 pt-1 border-t border-slate-200 flex items-center justify-between text-[8px] font-mono">
              <div class="flex items-center gap-1">
                <span class="font-bold text-slate-600">VIP Queue:</span>
                <span v-if="starvationQueue.length === 0" class="text-slate-400">Empty ✓</span>
                <span
                  v-for="task in starvationQueue" :key="task"
                  class="bg-rose-600 text-white px-1 rounded animate-pulse font-bold"
                >
                  {{ task }}
                </span>
              </div>
              <div class="flex items-center gap-1">
                <span class="font-bold text-slate-600">Browser Display:</span>
                <span v-if="s < 19" class="text-emerald-600 font-bold">60 FPS</span>
                <span v-else-if="s >= 19 && s < 21" class="text-rose-600 font-black animate-pulse">0 FPS (DEADLOCK)</span>
                <span v-else class="text-emerald-700 font-black">60 FPS (SMOOTH)</span>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
