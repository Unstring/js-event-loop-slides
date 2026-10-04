<script setup lang="ts">
import { computed } from 'vue'
const props = defineProps<{ step?: number }>()
const s = computed(() => Math.min(Math.max(props.step ?? 0, 0), 22))

// Multi-threaded scenario steps
const multiPhases = [
  { id: 1, label: 'Thread A starts', action: 'reads #box', color: 'bg-sky-500', who: 'A' },
  { id: 2, label: 'Thread B starts', action: 'deletes #box', color: 'bg-rose-500', who: 'B' },
  { id: 3, label: 'Thread A continues', action: 'writes to deleted #box', color: 'bg-red-700', who: 'A', crash: true },
]

// Single-threaded event loop steps
const singlePhases = [
  { id: 13, label: 'Script runs on main thread', action: 'reads and writes #box safely', color: 'bg-sky-500' },
  { id: 15, label: 'setTimeout callback queues', action: 'waits in Macrotask queue', color: 'bg-amber-500' },
  { id: 17, label: 'Script finishes', action: 'call stack empty', color: 'bg-slate-400' },
  { id: 19, label: 'Event Loop dequeues', action: 'runs setTimeout callback', color: 'bg-emerald-600' },
  { id: 21, label: 'No race condition', action: '#box is always safe ✓', color: 'bg-green-600' },
]

const insightStep = computed(() => {
  if (s.value >= 21) return 'safe'
  if (s.value >= 13) return 'single'
  if (s.value >= 7) return 'crash'
  if (s.value >= 1) return 'multi'
  return 'none'
})
</script>

<template>
  <div class="h-full flex flex-col gap-2 select-none">
    <!-- Header -->
    <div class="flex items-center gap-3 pb-1 border-b-2 border-slate-200">
      <span class="text-2xl">⚡</span>
      <div>
        <h2 class="text-xl font-black text-slate-900 leading-tight">Thread Model: Multi-Thread vs Event Loop</h2>
        <p class="text-xs text-slate-500">Chapter 1 of 5 · Interactive Diagram</p>
      </div>
      <div class="ml-auto px-2 py-1 rounded-lg bg-sky-600 text-white text-xs font-bold">Step {{ s }}/22</div>
    </div>

    <div class="grid grid-cols-2 gap-4 flex-1">
      <!-- LEFT: Multi-threaded danger zone -->
      <div class="flex flex-col">
        <div class="text-xs font-black text-rose-700 uppercase tracking-wider mb-2">
          ❌ Multi-Threaded (e.g. Java)
        </div>

        <!-- Threads -->
        <div class="flex gap-2 mb-2">
          <div
            class="flex-1 border-2 rounded-xl p-2 transition-all duration-400"
            :class="s >= 1 ? 'bg-sky-50 border-sky-400 opacity-100' : 'opacity-0'"
          >
            <div class="text-xs font-black text-sky-800 mb-1">🔵 Thread A</div>
            <div class="text-[10px] font-mono text-sky-700">
              <div :class="s >= 1 ? 'opacity-100' : 'opacity-0'" style="transition: opacity 0.3s">const box = getElementById()</div>
              <div :class="s >= 4 ? 'opacity-100' : 'opacity-0 blur-sm'" style="transition: all 0.3s delay-100ms">box.style.width = '200px'</div>
              <div v-if="s >= 7" class="mt-1 text-red-600 font-black text-[10px] animate-pulse">
                💥 NULL POINTER ERROR!
              </div>
            </div>
          </div>
          <div
            class="flex-1 border-2 rounded-xl p-2 transition-all duration-400"
            :class="s >= 2 ? 'bg-rose-50 border-rose-400 opacity-100' : 'opacity-0'"
          >
            <div class="text-xs font-black text-rose-800 mb-1">🔴 Thread B</div>
            <div class="text-[10px] font-mono text-rose-700">
              <div :class="s >= 2 ? 'opacity-100' : 'opacity-0'" style="transition: opacity 0.3s">// runs simultaneously</div>
              <div :class="s >= 5 ? 'opacity-100' : 'opacity-0'" style="transition: opacity 0.3s delay-100ms">box.remove() // ⚠️ DELETES box</div>
            </div>
          </div>
        </div>

        <!-- DOM box (gets destroyed) -->
        <div class="relative border-2 rounded-xl p-3 text-center mb-2 transition-all duration-500"
          :class="s >= 7
            ? 'bg-red-50 border-red-500 scale-95'
            : s >= 1
            ? 'bg-slate-50 border-slate-300'
            : 'opacity-0'"
        >
          <div class="text-xs font-bold text-slate-700 mb-1">DOM: #box</div>
          <div
            class="w-16 h-8 mx-auto rounded border-2 border-sky-400 bg-sky-100 flex items-center justify-center text-xs font-bold text-sky-800 transition-all duration-500"
            :class="s >= 6 ? 'opacity-0 scale-0' : 'opacity-100 scale-100'"
          >#box</div>
          <div v-if="s >= 7" class="absolute inset-0 flex items-center justify-center text-red-600 font-black text-sm rounded-xl">
            💥 CRASHED
          </div>
        </div>

        <!-- Multi-thread problems -->
        <div v-if="s >= 9" class="bg-red-50 border-2 border-red-300 rounded-xl p-2">
          <div class="text-xs font-black text-red-800 mb-1">Problems with Multi-Threading + DOM:</div>
          <div class="space-y-0.5 text-[10px] text-red-900">
            <div :class="s >= 9 ? 'opacity-100' : 'opacity-0'" style="transition: opacity 0.3s">⚠️ Race conditions crash the browser</div>
            <div :class="s >= 10 ? 'opacity-100' : 'opacity-0'" style="transition: opacity 0.3s delay-100ms">🔒 Mutexes cause deadlocks & UI freezes</div>
            <div :class="s >= 11 ? 'opacity-100' : 'opacity-0'" style="transition: opacity 0.3s delay-200ms">🐛 Heisenbugs: bugs that vanish when you look</div>
            <div :class="s >= 12 ? 'opacity-100' : 'opacity-0'" style="transition: opacity 0.3s delay-300ms">📈 Complexity grows O(n²) with thread count</div>
          </div>
        </div>
      </div>

      <!-- RIGHT: Single-threaded safety -->
      <div class="flex flex-col">
        <div class="text-xs font-black text-emerald-700 uppercase tracking-wider mb-2">
          ✅ Single-Threaded + Event Loop (JS)
        </div>

        <!-- The runtime stack visual -->
        <div class="relative flex flex-col gap-1.5 mb-2">
          <div
            v-for="(phase, i) in singlePhases" :key="i"
            class="border-2 rounded-lg px-3 py-1.5 flex items-center gap-2 transition-all duration-400"
            :class="[
              s >= phase.id ? 'opacity-100 translate-x-0' : 'opacity-0 translate-x-6',
              phase.id === 13 ? 'bg-sky-50 border-sky-300' :
              phase.id === 15 ? 'bg-amber-50 border-amber-300' :
              phase.id === 17 ? 'bg-slate-50 border-slate-300' :
              phase.id === 19 ? 'bg-emerald-50 border-emerald-300' :
              'bg-green-50 border-green-400'
            ]"
            style="transition: opacity 0.35s, transform 0.35s"
          >
            <div :class="[phase.color, 'w-2 h-2 rounded-full shrink-0']"></div>
            <div>
              <div class="text-xs font-bold text-slate-800">{{ phase.label }}</div>
              <div class="text-[10px] text-slate-600">{{ phase.action }}</div>
            </div>
          </div>
        </div>

        <!-- DOM box (stays safe) -->
        <div
          class="border-2 rounded-xl p-3 text-center mb-2 transition-all duration-500"
          :class="s >= 13 ? 'bg-emerald-50 border-emerald-400 opacity-100' : 'opacity-0'"
        >
          <div class="text-xs font-bold text-slate-700 mb-1">DOM: #box (always safe)</div>
          <div
            class="w-16 h-8 mx-auto rounded border-2 border-emerald-500 bg-emerald-100 flex items-center justify-center text-xs font-bold text-emerald-900 transition-all duration-500"
            :class="s >= 21 ? 'border-green-600 bg-green-100 scale-110' : ''"
          >
            {{ s >= 21 ? '✓ safe' : '#box' }}
          </div>
        </div>

        <!-- Final summary box -->
        <div
          class="mt-auto bg-sky-50 border-2 border-sky-400 rounded-xl p-2.5 transition-all duration-500"
          :class="s >= 22 ? 'opacity-100' : 'opacity-0'"
        >
          <div class="text-xs font-black text-sky-900 mb-1">🏆 Why It Works</div>
          <div class="text-[11px] text-sky-900">
            JS runs on <strong>one thread</strong>. The host environment handles blocking work on separate OS threads. Results are safely handed back via the <strong>callback queue</strong> — no mutex, no deadlock, no crash.
          </div>
        </div>
      </div>
    </div>

    <!-- Bottom question strip -->
    <div
      class="bg-slate-900 rounded-xl px-3 py-1.5 text-center transition-all duration-500"
      :class="s >= 20 ? 'opacity-100' : 'opacity-0'"
    >
      <span class="text-white text-xs font-bold">❓ Q: If JS is single-threaded, how does setTimeout run while your code is executing?</span>
      <span v-if="s >= 22" class="text-amber-400 font-black text-xs ml-2">→ The Host Environment handles it on a different OS thread!</span>
    </div>
  </div>
</template>
