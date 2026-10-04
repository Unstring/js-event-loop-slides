<script setup lang="ts">
import { computed } from 'vue'

const props = defineProps<{ step?: number }>()
const s = computed(() => Math.min(Math.max(props.step ?? 0, 0), 22))

const multiPhases = [
  { id: 1, label: 'Thread A', action: 'const box = getElementById("box")', status: 'Reads #box', color: 'bg-sky-50 border-sky-300 text-sky-900' },
  { id: 3, label: 'Thread B', action: 'box.parentNode.removeChild(box)', status: 'Deletes node', color: 'bg-rose-50 border-rose-300 text-rose-900' },
  { id: 5, label: 'Thread A', action: 'box.style.width = "200px"', status: 'Writes to null', color: 'bg-amber-50 border-amber-300 text-amber-900' },
  { id: 7, label: 'CPU CORE', action: 'MEMORY CORRUPTION / SEGFAULT', status: '💥 CRASH', color: 'bg-red-600 text-white font-bold border-red-700' },
]

const singlePhases = [
  { id: 11, label: 'Main Thread', action: 'script executes sequentially', status: 'Running', color: 'bg-sky-50 border-sky-300 text-sky-900' },
  { id: 13, label: 'Host Web API', action: 'timer ticks on background C++ thread', status: 'Offloaded', color: 'bg-emerald-50 border-emerald-300 text-emerald-900' },
  { id: 15, label: 'Task Queue', action: 'timer expires → callback queues', status: 'Waiting', color: 'bg-amber-50 border-amber-300 text-amber-900' },
  { id: 17, label: 'Event Loop', action: 'stack empty? → dequeues callback safely', status: 'Dispatched', color: 'bg-violet-50 border-violet-300 text-violet-900' },
]
</script>

<template>
  <div class="h-full flex flex-col justify-between select-none text-slate-800 text-xs">
    <!-- Header -->
    <div class="flex items-center gap-2 pb-1 border-b border-slate-200 shrink-0">
      <span class="text-xl">⚡</span>
      <div>
        <h2 class="text-base font-black text-slate-900 leading-tight">Thread Model: Multi-Thread vs Event Loop</h2>
        <p class="text-[10px] text-slate-500">Chapter 1 of 5 · Interactive Architecture Comparison</p>
      </div>
      <div class="ml-auto flex items-center gap-1.5">
        <span class="px-2 py-0.5 rounded bg-emerald-100 text-emerald-800 text-[10px] font-bold">Race-Condition Free</span>
        <div class="px-2 py-0.5 rounded bg-sky-600 text-white text-[10px] font-bold">Step {{ s }}/22</div>
      </div>
    </div>

    <!-- 2 Comparison Columns -->
    <div class="grid grid-cols-2 gap-2 flex-1 min-h-0 py-1">
      <!-- LEFT: Multi-Threaded Collision (Steps 1-10) -->
      <div
        class="border-2 rounded-xl p-2 bg-rose-50/30 flex flex-col justify-between transition-all duration-300"
        :class="s >= 1 ? 'border-rose-300' : 'border-slate-200 opacity-40'"
      >
        <div>
          <div class="flex items-center justify-between mb-1">
            <span class="text-[10px] font-black text-rose-800 uppercase tracking-wider flex items-center gap-1">
              <span>❌</span> Multi-Threaded Model (Java/C++)
            </span>
            <span class="text-[9px] font-bold text-rose-600 bg-rose-100 px-1.5 py-0.2 rounded">Steps 1-10</span>
          </div>

          <!-- Thread Execution Sequence -->
          <div class="flex flex-col gap-1 mb-2">
            <div
              v-for="p in multiPhases" :key="p.id"
              class="border rounded-md px-2 py-1 flex items-center justify-between text-[10px] font-mono transition-all duration-200"
              :class="[
                p.color,
                s >= p.id ? 'opacity-100 translate-x-0' : 'opacity-15 -translate-x-2'
              ]"
            >
              <div class="flex items-center gap-1.5 truncate">
                <span class="font-bold shrink-0">[{{ p.label }}]</span>
                <span class="truncate">{{ p.action }}</span>
              </div>
              <span class="text-[8px] px-1 rounded bg-black/10 shrink-0 font-sans font-bold">{{ p.status }}</span>
            </div>
          </div>

          <!-- DOM Target Box Visual -->
          <div class="border rounded-lg p-2 bg-white flex items-center justify-between text-[10px]">
            <div>
              <span class="font-bold text-slate-700">DOM Shared Memory:</span>
              <span class="font-mono text-slate-500 ml-1">&lt;div id="box"&gt;</span>
            </div>
            <div
              class="px-2 py-0.5 rounded text-[10px] font-bold transition-all duration-300"
              :class="s >= 7 ? 'bg-red-600 text-white animate-pulse' : (s >= 3 ? 'bg-amber-100 text-amber-900 border border-amber-300' : 'bg-slate-100 text-slate-700')"
            >
              {{ s >= 7 ? '💥 CRASHED' : (s >= 3 ? '⚠️ DELETED' : 'ALIVE') }}
            </div>
          </div>
        </div>

        <!-- Multi-thread hazards list -->
        <div
          class="border border-rose-200 rounded-lg p-1.5 bg-rose-100/60 text-[9px] text-rose-900 transition-all duration-300"
          :class="s >= 8 ? 'opacity-100' : 'opacity-0'"
        >
          <div class="font-bold text-[9px] mb-0.5">Why this fails for Web Browsers:</div>
          <div class="grid grid-cols-2 gap-x-2 gap-y-0.5">
            <div>• Race conditions crash UI</div>
            <div>• Mutex locks cause deadlocks</div>
            <div>• Complex thread synchronization</div>
            <div>• Non-deterministic UI rendering</div>
          </div>
        </div>
      </div>

      <!-- RIGHT: Single-Threaded Event Loop (Steps 11-19) -->
      <div
        class="border-2 rounded-xl p-2 bg-emerald-50/30 flex flex-col justify-between transition-all duration-300"
        :class="s >= 11 ? 'border-emerald-300' : 'border-slate-200 opacity-40'"
      >
        <div>
          <div class="flex items-center justify-between mb-1">
            <span class="text-[10px] font-black text-emerald-800 uppercase tracking-wider flex items-center gap-1">
              <span>✅</span> Single Thread + Event Loop (JS)
            </span>
            <span class="text-[9px] font-bold text-emerald-600 bg-emerald-100 px-1.5 py-0.2 rounded">Steps 11-19</span>
          </div>

          <!-- Event Loop Pipeline Steps -->
          <div class="flex flex-col gap-1 mb-2">
            <div
              v-for="p in singlePhases" :key="p.id"
              class="border rounded-md px-2 py-1 flex items-center justify-between text-[10px] font-mono transition-all duration-200"
              :class="[
                p.color,
                s >= p.id ? 'opacity-100 translate-x-0' : 'opacity-15 translate-x-2'
              ]"
            >
              <div class="flex items-center gap-1.5 truncate">
                <span class="font-bold shrink-0">[{{ p.label }}]</span>
                <span class="truncate">{{ p.action }}</span>
              </div>
              <span class="text-[8px] px-1 rounded bg-black/10 shrink-0 font-sans font-bold">{{ p.status }}</span>
            </div>
          </div>

          <!-- DOM Target Box Visual -->
          <div class="border rounded-lg p-2 bg-white flex items-center justify-between text-[10px]">
            <div>
              <span class="font-bold text-slate-700">DOM Shared Memory:</span>
              <span class="font-mono text-slate-500 ml-1">&lt;div id="box"&gt;</span>
            </div>
            <div
              class="px-2 py-0.5 rounded text-[10px] font-bold bg-emerald-100 text-emerald-900 border border-emerald-300 transition-all duration-300"
              :class="s >= 17 ? 'scale-105 ring-2 ring-emerald-400' : ''"
            >
              ✓ 100% THREAD SAFE
            </div>
          </div>
        </div>

        <!-- Single thread guarantees -->
        <div
          class="border border-emerald-200 rounded-lg p-1.5 bg-emerald-100/60 text-[9px] text-emerald-950 transition-all duration-300"
          :class="s >= 18 ? 'opacity-100' : 'opacity-0'"
        >
          <div class="font-bold text-[9px] mb-0.5">Why JavaScript Wins on the Web:</div>
          <div class="grid grid-cols-2 gap-x-2 gap-y-0.5">
            <div>• 1 thread controls DOM safely</div>
            <div>• Zero lock contention / deadlocks</div>
            <div>• Predictable state transitions</div>
            <div>• Run-to-completion guarantees</div>
          </div>
        </div>
      </div>
    </div>

    <!-- Bottom Question & Synthesis Strip (Steps 20-22) -->
    <div
      class="border rounded-lg px-2.5 py-1 text-slate-900 flex items-center justify-between shrink-0 transition-all duration-300"
      :class="s >= 20 ? 'bg-sky-50 border-sky-300 opacity-100' : 'bg-slate-50 border-slate-200 opacity-30'"
    >
      <div class="flex items-center gap-1.5 text-[10px]">
        <span class="font-black text-sky-800">❓ Key Question:</span>
        <span>"If JS is single-threaded, how can we fetch data without freezing the UI?"</span>
      </div>
      <div class="text-[10px] font-bold text-emerald-800 flex items-center gap-1">
        <span v-if="s >= 21">→ Answer:</span>
        <span v-if="s >= 22" class="underline">Web APIs handle the wait; Event Loop dispatches the result!</span>
      </div>
    </div>
  </div>
</template>
