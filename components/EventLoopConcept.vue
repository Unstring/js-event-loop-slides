<script setup lang="ts">
import { computed } from 'vue'
const props = defineProps<{ step?: number }>()
const s = computed(() => Math.min(Math.max(props.step ?? 0, 0), 22))

const phases = [
  { id: 7, emoji: '1️⃣', label: 'Check Stack', detail: 'Is the call stack empty?', color: 'bg-sky-100 border-sky-400 text-sky-900', arrow: true },
  { id: 9, emoji: '2️⃣', label: 'Drain Microtasks', detail: 'Run ALL microtasks to completion (Promise callbacks, queueMicrotask)', color: 'bg-violet-100 border-violet-400 text-violet-900', arrow: true },
  { id: 11, emoji: '3️⃣', label: 'Render Opportunity', detail: 'If V-Sync pulse (16.6ms): run rAF → Style → Layout → Paint → Composite', color: 'bg-rose-100 border-rose-400 text-rose-900', arrow: true },
  { id: 13, emoji: '4️⃣', label: 'ONE Macrotask', detail: 'Dequeue exactly one task (setTimeout, click, I/O) and execute it', color: 'bg-amber-100 border-amber-400 text-amber-900', arrow: false },
]

const envComparison = [
  {
    id: 14,
    name: 'Browser',
    emoji: '🌐',
    color: 'bg-sky-50 border-sky-300',
    items: [
      { id: 14, text: 'Web APIs: Timer, Fetch, DOM, Audio, WebGL' },
      { id: 15, text: 'Chromium C++ threads handle timers & network' },
      { id: 16, text: 'requestAnimationFrame syncs to 60/120Hz GPU' },
    ]
  },
  {
    id: 17,
    name: 'Node.js',
    emoji: '🦺',
    color: 'bg-emerald-50 border-emerald-300',
    items: [
      { id: 17, text: 'libuv: event demultiplexer (epoll/kqueue/IOCP)' },
      { id: 18, text: 'Worker Pool: 4 threads for file I/O, DNS, crypto' },
      { id: 19, text: 'No render phase — extra phase: setImmediate' },
    ]
  },
]
</script>

<template>
  <div class="h-full flex flex-col gap-2 select-none">
    <div class="flex items-center gap-3 pb-1 border-b-2 border-slate-200">
      <span class="text-2xl">↻</span>
      <div>
        <h2 class="text-xl font-black text-slate-900 leading-tight">The Event Loop</h2>
        <p class="text-xs text-slate-500">Chapter 3 of 5 · Concept</p>
      </div>
      <div class="ml-auto px-2 py-1 rounded-lg bg-sky-600 text-white text-xs font-bold">Step {{ s }}/22</div>
    </div>

    <!-- What is the event loop -->
    <div class="grid grid-cols-2 gap-3 flex-1">
      <div class="flex flex-col gap-2">
        <div class="text-xs font-black text-slate-700 uppercase tracking-wider">📌 What is the Event Loop?</div>

        <div
          class="border-2 border-slate-300 rounded-xl p-3 text-sm font-medium text-slate-800 bg-slate-50 leading-relaxed transition-all duration-400"
          :class="s >= 1 ? 'opacity-100' : 'opacity-0'"
        >
          The Event Loop is a C++ <strong>infinite loop</strong> inside V8/Node.js that continuously asks:
          <em class="text-sky-700 font-bold">"Is the call stack empty AND is there work to do?"</em>
        </div>

        <div
          class="bg-slate-900 rounded-xl p-3 font-mono text-[11px] leading-relaxed transition-all duration-400"
          :class="s >= 3 ? 'opacity-100' : 'opacity-0'"
        >
          <div class="text-slate-500 text-[10px] mb-1">// Pseudocode — actual C++ inside V8</div>
          <div :class="s >= 3 ? 'text-amber-300' : 'text-slate-600'">while (true) {</div>
          <div class="pl-4" :class="s >= 4 ? 'text-sky-300' : 'text-slate-600 opacity-0'" style="transition: opacity 0.3s">if (stack.isEmpty()) {</div>
          <div class="pl-8" :class="s >= 5 ? 'text-violet-300' : 'text-slate-600 opacity-0'" style="transition: opacity 0.3s">drainMicrotasks()</div>
          <div class="pl-8" :class="s >= 5 ? 'text-rose-300' : 'text-slate-600 opacity-0'" style="transition: opacity 0.3s">checkRender()</div>
          <div class="pl-8" :class="s >= 6 ? 'text-amber-300' : 'text-slate-600 opacity-0'" style="transition: opacity 0.3s">runOneMacrotask()</div>
          <div class="pl-4" :class="s >= 4 ? 'text-sky-300' : 'text-slate-600 opacity-0'" style="transition: opacity 0.3s">}</div>
          <div :class="s >= 3 ? 'text-amber-300' : 'text-slate-600'">}</div>
        </div>

        <!-- The 4 phases -->
        <div class="text-xs font-black text-slate-700 uppercase tracking-wider" :class="s >= 7 ? 'opacity-100' : 'opacity-0'">⚙️ The 4-Phase Heartbeat</div>
        <div class="flex flex-col gap-1.5">
          <div
            v-for="phase in phases" :key="phase.id"
            class="border-2 rounded-xl px-3 py-1.5 flex items-center gap-2 text-xs font-bold transition-all duration-400"
            :class="[phase.color, s >= phase.id ? 'opacity-100 translate-x-0' : 'opacity-0 -translate-x-4']"
            style="transition: opacity 0.35s, transform 0.35s"
          >
            <span class="text-base shrink-0">{{ phase.emoji }}</span>
            <div class="flex-1">
              <div class="font-black">{{ phase.label }}</div>
              <div class="text-[10px] font-normal opacity-80">{{ phase.detail }}</div>
            </div>
            <span v-if="phase.arrow" class="text-slate-400 shrink-0">→</span>
          </div>
        </div>
      </div>

      <!-- RIGHT: Host environments + recap -->
      <div class="flex flex-col gap-2">
        <div class="text-xs font-black text-slate-700 uppercase tracking-wider">🏗️ Host Environments</div>

        <div
          v-for="env in envComparison" :key="env.name"
          class="border-2 rounded-xl p-2.5 transition-all duration-400"
          :class="[env.color, s >= env.id ? 'opacity-100' : 'opacity-0']"
        >
          <div class="flex items-center gap-2 mb-1.5">
            <span class="text-lg">{{ env.emoji }}</span>
            <span class="font-black text-sm text-slate-900">{{ env.name }}</span>
          </div>
          <div class="space-y-0.5">
            <div
              v-for="item in env.items" :key="item.id"
              class="text-[11px] text-slate-700 flex items-start gap-1.5 transition-all duration-300"
              :class="s >= item.id ? 'opacity-100' : 'opacity-0'"
            >
              <span class="text-slate-400 mt-0.5 shrink-0">•</span>
              {{ item.text }}
            </div>
          </div>
        </div>

        <!-- Key question -->
        <div
          class="bg-amber-50 border-2 border-amber-400 rounded-xl px-3 py-2 mt-auto transition-all duration-500"
          :class="s >= 20 ? 'opacity-100' : 'opacity-0'"
        >
          <div class="text-xs font-black text-amber-900 mb-1">❓ Why does the order matter?</div>
          <div class="text-[11px] text-amber-900 space-y-0.5">
            <div :class="s >= 20 ? 'opacity-100' : 'opacity-0'" style="transition: opacity 0.3s">🔑 Microtasks <strong>always</strong> run before macrotasks</div>
            <div :class="s >= 21 ? 'opacity-100' : 'opacity-0'" style="transition: opacity 0.3s delay-100ms">🎨 Render only happens if the display needs updating</div>
            <div :class="s >= 22 ? 'opacity-100' : 'opacity-0'" style="transition: opacity 0.3s delay-200ms">⏱️ Only <strong>ONE</strong> macrotask per loop turn — ensures fairness</div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
