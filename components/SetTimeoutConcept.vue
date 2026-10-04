<script setup lang="ts">
import { computed } from 'vue'

const props = defineProps<{ step?: number }>()
const s = computed(() => Math.min(Math.max(props.step ?? 0, 0), 22))

const whatItDoes = [
  { id: 1, text: 'NOT a sleep() — JavaScript thread is NEVER blocked.', icon: '❌', bad: true },
  { id: 2, text: 'Registers a timer in the Host Environment (C++ / libuv).', icon: '📞', bad: false },
  { id: 3, text: 'Host environment monitors timer on a separate OS thread.', icon: '🧵', bad: false },
  { id: 4, text: 'When delay expires → callback pushed to Macrotask Queue.', icon: '📬', bad: false },
  { id: 5, text: 'Runs ONLY when Call Stack is empty & microtasks drained.', icon: '⏳', bad: false },
  { id: 6, text: 'Specified delay is a MINIMUM wait, never guaranteed exact.', icon: '⚠️', bad: true },
]

const heapItems = [
  { id: 8, timer: 'Timer A (10ms)', expires: 'T+10ms', pos: 'Root (Soonest)', color: 'bg-emerald-100 border-emerald-400 text-emerald-950 font-bold' },
  { id: 9, timer: 'Timer B (50ms)', expires: 'T+50ms', pos: 'Child Node', color: 'bg-sky-100 border-sky-300 text-sky-950' },
  { id: 10, timer: 'Timer C (100ms)', expires: 'T+100ms', pos: 'Leaf Node', color: 'bg-violet-100 border-violet-300 text-violet-950' },
]

const clampingRules = [
  { id: 18, text: 'First 4 nested calls: 0ms delay allowed', tag: '0ms OK', color: 'bg-emerald-50 border-emerald-300 text-emerald-900' },
  { id: 19, text: 'Nesting depth ≥ 5: Clamped to 4ms minimum (HTML5 spec)', tag: '4ms Clamp', color: 'bg-amber-50 border-amber-300 text-amber-900' },
  { id: 20, text: 'Background / hidden tab: Throttled to 1000ms (1 sec)', tag: 'Battery Saver', color: 'bg-orange-50 border-orange-300 text-orange-900' },
  { id: 21, text: 'requestAnimationFrame: Suspended entirely in hidden tabs', tag: 'GPU Paused', color: 'bg-rose-50 border-rose-300 text-rose-900' },
]
</script>

<template>
  <div class="h-full flex flex-col justify-between select-none text-slate-800 text-xs">
    <!-- Header -->
    <div class="flex items-center gap-2 pb-1 border-b border-slate-200 shrink-0">
      <span class="text-xl">⏱️</span>
      <div>
        <h2 class="text-base font-black text-slate-900 leading-tight">setTimeout Internals: Host Timers & The 0ms Myth</h2>
        <p class="text-[10px] text-slate-500">Chapter 4 of 5 · Min-Heap Mechanics & HTML5 Clamping</p>
      </div>
      <div class="ml-auto flex items-center gap-1.5">
        <span class="px-2 py-0.5 rounded bg-sky-100 text-sky-900 text-[10px] font-bold">Host Priority Queue</span>
        <div class="px-2 py-0.5 rounded bg-sky-600 text-white text-[10px] font-bold">Step {{ s }}/22</div>
      </div>
    </div>

    <!-- 2 Columns -->
    <div class="grid grid-cols-2 gap-2 flex-1 min-h-0 py-1">
      <!-- LEFT: What It Does (Steps 1-6) + The 0ms Myth (Steps 13-17) -->
      <div class="flex flex-col gap-1.5 justify-between min-h-0">
        <!-- What setTimeout Actually Does (Steps 1-6) -->
        <div class="border border-slate-200 rounded-lg p-2 bg-slate-50 flex flex-col gap-1">
          <div class="text-[10px] font-black text-slate-700 uppercase tracking-wider flex justify-between">
            <span>📌 What setTimeout Actually Does</span>
            <span class="text-[9px] text-slate-400 font-mono">Steps 1-6</span>
          </div>
          <div class="flex flex-col gap-0.5">
            <div
              v-for="item in whatItDoes" :key="item.id"
              class="border rounded px-1.5 py-0.5 text-[9px] flex items-center gap-1.5 transition-all duration-200"
              :class="[
                item.bad ? 'bg-rose-50/70 border-rose-200 text-rose-900 font-medium' : 'bg-white border-slate-200 text-slate-800',
                s >= item.id ? 'opacity-100 translate-x-0' : 'opacity-15 -translate-x-2'
              ]"
            >
              <span class="shrink-0 text-[10px]">{{ item.icon }}</span>
              <span class="truncate">{{ item.text }}</span>
            </div>
          </div>
        </div>

        <!-- The 0ms Myth Code Box (Steps 13-17) -->
        <div
          class="border border-rose-300 rounded-lg p-2 bg-slate-900 text-slate-100 flex flex-col gap-1 transition-all duration-300"
          :class="s >= 13 ? 'opacity-100' : 'opacity-20'"
        >
          <div class="text-[10px] font-black text-rose-400 uppercase tracking-wider flex justify-between">
            <span>💥 The "0ms" Myth Exposed</span>
            <span class="text-[9px] text-amber-400 font-mono">Steps 13-17</span>
          </div>
          <div class="font-mono text-[9px] flex flex-col gap-0.5">
            <div class="text-slate-400">// What developers think:</div>
            <div :class="s >= 13 ? 'text-rose-300 font-bold' : 'text-slate-600'">setTimeout(fn, 0); // "runs immediately" (FALSE!)</div>
            <div class="text-slate-400 mt-1">// What the JavaScript runtime actually does:</div>
            <div :class="s >= 14 ? 'text-amber-300' : 'text-slate-600'">1. Host registers timer at T+0ms (Root of Min-Heap)</div>
            <div :class="s >= 15 ? 'text-amber-300' : 'text-slate-600'">2. Timer expires → moves fn to Macrotask Queue</div>
            <div :class="s >= 16 ? 'text-amber-300' : 'text-slate-600'">3. MUST wait for Call Stack to be 100% empty</div>
            <div :class="s >= 17 ? 'text-emerald-300 font-bold' : 'text-slate-600'">4. MUST wait for ALL Microtasks to drain first!</div>
          </div>
        </div>
      </div>

      <!-- RIGHT: Min-Heap Queue (Steps 7-12) + Clamping Rules (Steps 18-22) -->
      <div class="flex flex-col gap-1.5 justify-between min-h-0">
        <!-- Min-Heap Priority Queue (Steps 7-12) -->
        <div
          class="border border-slate-200 rounded-lg p-2 bg-slate-50 flex flex-col gap-1 transition-all duration-300"
          :class="s >= 7 ? 'opacity-100' : 'opacity-25'"
        >
          <div class="text-[10px] font-black text-slate-700 uppercase tracking-wider flex justify-between">
            <span>🌳 Host Min-Heap Priority Queue</span>
            <span class="text-[9px] text-slate-400 font-mono">Steps 7-12</span>
          </div>
          <div class="text-[9px] text-slate-600 leading-tight">
            Timers are stored in a binary Min-Heap inside C++/libuv. O(log n) insert; root is always the soonest expiring timer.
          </div>

          <div class="flex flex-col gap-1 mt-0.5">
            <div
              v-for="item in heapItems" :key="item.id"
              class="border rounded px-2 py-0.5 flex items-center justify-between text-[9px] transition-all duration-200"
              :class="[item.color, s >= item.id ? 'opacity-100 scale-100' : 'opacity-20 scale-98']"
            >
              <div>
                <span class="font-bold">{{ item.timer }}</span>
                <span class="text-[8px] opacity-75 ml-1">({{ item.pos }})</span>
              </div>
              <span class="font-mono font-bold">{{ item.expires }}</span>
            </div>
          </div>
          <div class="text-[8px] text-slate-500 mt-0.5" :class="s >= 11 ? 'opacity-100' : 'opacity-0'">
            OS sets 1 hardware interrupt for root. When it fires, OS sends callback to Event Loop.
          </div>
        </div>

        <!-- HTML5 Clamping & Tab Throttling (Steps 18-21) -->
        <div
          class="border border-amber-200 rounded-lg p-1.5 bg-amber-50/50 flex flex-col gap-1 transition-all duration-300 shrink-0"
          :class="s >= 18 ? 'opacity-100' : 'opacity-25'"
        >
          <div class="text-[10px] font-black text-amber-900 uppercase tracking-wider flex justify-between">
            <span>📋 HTML5 Clamping & Tab Throttling Rules</span>
            <span class="text-[9px] text-amber-700 font-mono">Steps 18-21</span>
          </div>
          <div class="grid grid-cols-2 gap-1 text-[9px]">
            <div
              v-for="r in clampingRules" :key="r.id"
              class="border rounded p-1 flex flex-col justify-between transition-all duration-200"
              :class="[r.color, s >= r.id ? 'opacity-100' : 'opacity-20']"
            >
              <span class="leading-tight text-[8px]">{{ r.text }}</span>
              <span class="text-[7px] font-bold font-mono px-1 rounded bg-black/10 self-start mt-0.5">{{ r.tag }}</span>
            </div>
          </div>
        </div>

        <!-- Golden Takeaway (Step 22) -->
        <div
          class="border border-emerald-300 rounded-lg p-1.5 bg-emerald-50 text-emerald-950 transition-all duration-300 shrink-0"
          :class="s >= 22 ? 'opacity-100 translate-y-0' : 'opacity-0 translate-y-1'"
        >
          <span class="font-bold text-[9px]">💡 The Golden Law of Timers:</span>
          <span class="text-[8px] ml-1">
            <code class="bg-white px-1 rounded border border-emerald-200">setTimeout(fn, d)</code> guarantees a <strong>minimum delay</strong> before entering the queue, NEVER exact execution time!
          </span>
        </div>
      </div>
    </div>
  </div>
</template>
