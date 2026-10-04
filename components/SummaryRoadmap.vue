<script setup lang="ts">
import { computed } from 'vue'

const props = defineProps<{ step?: number }>()
const s = computed(() => Math.min(Math.max(props.step ?? 0, 0), 22))

const pillars = [
  {
    id: 2,
    num: '01',
    title: 'Single-Threaded JS',
    icon: '🧵',
    color: 'border-sky-300 bg-sky-50 text-sky-950',
    tag: 'Safety',
    takeaway: '1 Call Stack, 1 Memory Heap. Prevents DOM race conditions & deadlocks. Run-to-completion guarantees.',
  },
  {
    id: 5,
    num: '02',
    title: 'Call Stack & Context',
    icon: '⚡',
    color: 'border-blue-300 bg-blue-50 text-blue-950',
    tag: 'LIFO',
    takeaway: 'Frames push on function call, pop on return. Synchronous execution is blocking and uninterrupted.',
  },
  {
    id: 8,
    num: '03',
    title: 'Host APIs & Timers',
    icon: '🌐',
    color: 'border-emerald-300 bg-emerald-50 text-emerald-950',
    tag: 'Offloading',
    takeaway: 'Browser C++ / Node.js libuv OS threads handle timers (Min-Heap) and I/O in parallel without blocking JS.',
  },
  {
    id: 11,
    num: '04',
    title: 'The Event Loop',
    icon: '↻',
    color: 'border-amber-300 bg-amber-50 text-amber-950',
    tag: 'Heartbeat',
    takeaway: 'Orchestrator: Call Stack empty? → Drain Microtasks → Check 60fps Render → Pick ONE Macrotask.',
  },
  {
    id: 14,
    num: '05',
    title: 'Microtasks vs Macrotasks',
    icon: '⚖️',
    color: 'border-violet-300 bg-violet-50 text-violet-950',
    tag: 'Two-Tier',
    takeaway: 'Microtasks (Promises, queueMicrotask) drain 100% first. Macrotasks (timers, clicks) run 1 per turn.',
  },
  {
    id: 17,
    num: '06',
    title: 'Promises & async/await',
    icon: '🤝',
    color: 'border-indigo-300 bg-indigo-50 text-indigo-950',
    tag: 'Coroutines',
    takeaway: 'async/await is compiler desugaring. await suspends to Heap; microtask resumes function seamlessly.',
  },
]

const goldenRules = [
  { id: 19, rule: 'Never block the stack: Heavy CPU tasks freeze clicks, animations, and renders.' },
  { id: 20, rule: 'setTimeout(fn, 0) specifies minimum wait, not actual execution time.' },
  { id: 21, rule: 'Avoid microtask starvation: recursive Promises will permanently freeze the UI.' },
  { id: 22, rule: 'Execution order: Synchronous Code → Microtasks → Render Pass → Next Macrotask.' },
]
</script>

<template>
  <div class="h-full flex flex-col gap-1.5 select-none text-slate-800">
    <!-- Header -->
    <div class="flex items-center gap-3 pb-1 border-b-2 border-slate-200">
      <span class="text-2xl">🗺️</span>
      <div>
        <h2 class="text-xl font-black text-slate-900 leading-tight">The Complete Event Loop Mental Model</h2>
        <p class="text-xs text-slate-500">Grand Synthesis · All 5 Learning Outcomes United</p>
      </div>
      <div class="ml-auto px-2 py-1 rounded-lg bg-sky-600 text-white text-xs font-bold">Step {{ s }}/22</div>
    </div>

    <!-- Active Banner -->
    <div class="bg-slate-50 border border-slate-200 rounded-lg px-3 py-1 text-xs text-slate-700 flex items-center justify-between">
      <span class="font-medium">
        Everything you need to master asynchronous JavaScript in a single unified architectural blueprint.
      </span>
      <span class="text-[10px] text-emerald-700 font-bold bg-emerald-100 px-2 py-0.5 rounded">Core Mental Model</span>
    </div>

    <!-- Main Grid: 6 Pillars -->
    <div class="grid grid-cols-3 gap-2 flex-1 min-h-0">
      <div
        v-for="p in pillars" :key="p.id"
        class="border-2 rounded-xl p-2.5 flex flex-col justify-between transition-all duration-300 shadow-sm"
        :class="[
          p.color,
          s >= p.id ? 'opacity-100 scale-100 translate-y-0' : 'opacity-0 scale-95 translate-y-3'
        ]"
      >
        <div>
          <div class="flex items-center justify-between mb-1">
            <span class="text-lg">{{ p.icon }}</span>
            <div class="flex items-center gap-1">
              <span class="text-[8px] font-black uppercase px-1.5 py-0.2 rounded bg-black/10">{{ p.tag }}</span>
              <span class="font-mono text-[9px] font-bold opacity-60">#{{ p.num }}</span>
            </div>
          </div>
          <div class="font-black text-xs mb-1">{{ p.title }}</div>
          <div class="text-[10px] leading-snug opacity-90">{{ p.takeaway }}</div>
        </div>

        <div class="mt-2 pt-1 border-t border-black/10 flex items-center justify-between text-[8px] opacity-70">
          <span>Mastery Verified</span>
          <span>✓</span>
        </div>
      </div>
    </div>

    <!-- Bottom: 4 Golden Rules -->
    <div
      class="border-2 border-slate-300 rounded-xl p-2 bg-slate-900 text-slate-100 transition-all duration-400"
      :class="s >= 18 ? 'opacity-100 translate-y-0' : 'opacity-0 translate-y-2'"
    >
      <div class="text-[10px] font-black text-amber-400 uppercase tracking-wider mb-1 flex items-center gap-1.5">
        <span>⭐</span> The 4 Golden Laws of the JavaScript Runtime
      </div>
      <div class="grid grid-cols-2 gap-x-3 gap-y-0.5 text-[9px] text-slate-300 font-mono">
        <div
          v-for="g in goldenRules" :key="g.id"
          class="flex items-start gap-1 transition-opacity duration-300"
          :class="s >= g.id ? 'opacity-100 text-emerald-300' : 'opacity-30'"
        >
          <span class="text-amber-400 select-none">▶</span>
          <span>{{ g.rule }}</span>
        </div>
      </div>
    </div>
  </div>
</template>
