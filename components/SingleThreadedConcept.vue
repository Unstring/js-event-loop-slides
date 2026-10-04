<script setup lang="ts">
import { computed } from 'vue'
const props = defineProps<{ step?: number }>()
const s = computed(() => Math.min(Math.max(props.step ?? 0, 0), 22))

const whatIsThread = [
  { id: 1, text: 'A thread = one sequential instruction pointer in a CPU core', color: 'bg-sky-50 border-sky-300 text-sky-900' },
  { id: 2, text: 'Multi-threaded: many pointers running simultaneously on different cores', color: 'bg-violet-50 border-violet-300 text-violet-900' },
  { id: 3, text: 'Java, C++, Go = multi-threaded by default', color: 'bg-slate-50 border-slate-300 text-slate-700' },
  { id: 4, text: 'JavaScript = intentionally ONE thread (the main thread)', color: 'bg-amber-50 border-amber-400 text-amber-900' },
  { id: 5, text: 'Reason: DOM manipulation from two threads simultaneously → crash 💥', color: 'bg-rose-50 border-rose-400 text-rose-900' },
]

const domRaceLines = [
  { id: 1, thread: 'Thread A', code: 'const box = document.getElementById("box")', color: 'text-sky-700 bg-sky-50', label: 'Read' },
  { id: 2, thread: 'Thread B', code: 'box.parentNode.removeChild(box)  // box is GONE', color: 'text-rose-700 bg-rose-50', label: 'Delete' },
  { id: 3, thread: 'Thread A', code: 'box.style.width = "200px"  // 💥 NULL POINTER', color: 'text-red-700 bg-red-100 font-bold', label: 'CRASH' },
]

const solution = [
  { id: 1, step: 'Only 1 thread touches the DOM — ever', ok: true },
  { id: 2, step: 'Long tasks are offloaded to host APIs (timers, fetch, I/O)', ok: true },
  { id: 3, step: 'Callbacks queue up and run one at a time on the main thread', ok: true },
  { id: 4, step: 'Result: safe, predictable, no mutexes needed', ok: true },
]

const env = [
  { name: 'V8 Engine', detail: 'Parses + executes JS\nSingle-threaded', color: 'bg-sky-100 border-sky-400', emoji: '⚡' },
  { name: 'Browser Host', detail: 'C++ threads for timers\nnetwork, audio, GPU', color: 'bg-emerald-100 border-emerald-400', emoji: '🌐' },
  { name: 'Node.js / libuv', detail: 'Event demultiplexer\n4-thread worker pool', color: 'bg-violet-100 border-violet-400', emoji: '🦺' },
]
</script>

<template>
  <div class="h-full flex flex-col gap-2 select-none">
    <!-- Header -->
    <div class="flex items-center gap-3 pb-1 border-b-2 border-slate-200">
      <span class="text-2xl">🧵</span>
      <div>
        <h2 class="text-xl font-black text-slate-900 leading-tight">Why is JavaScript Single-Threaded?</h2>
        <p class="text-xs text-slate-500">Chapter 1 of 5 · Concept</p>
      </div>
      <div class="ml-auto px-2 py-1 rounded-lg bg-sky-600 text-white text-xs font-bold">Step {{ s }}/22</div>
    </div>

    <!-- Section 1: What is a thread -->
    <div class="grid grid-cols-2 gap-3 flex-1">
      <div class="flex flex-col gap-2">
        <div class="text-xs font-black text-slate-700 uppercase tracking-wider mb-1">📌 What is a Thread?</div>
        <div
          v-for="item in whatIsThread" :key="item.id"
          class="border-2 rounded-lg px-3 py-1.5 text-xs font-medium transition-all duration-400 leading-snug"
          :class="[item.color, s >= item.id ? 'opacity-100 translate-x-0' : 'opacity-0 -translate-x-4']"
          style="transition: opacity 0.35s, transform 0.35s"
        >
          <span class="font-black mr-1">{{ item.id }}.</span>{{ item.text }}
        </div>

        <!-- DOM Race Condition -->
        <div class="mt-2" :class="s >= 11 ? 'opacity-100' : 'opacity-0'" style="transition: opacity 0.4s">
          <div class="text-xs font-black text-rose-700 uppercase tracking-wider mb-1">💥 DOM Race Condition</div>
          <div class="bg-slate-900 rounded-lg p-2 font-mono text-xs leading-relaxed">
            <div
              v-for="(line, i) in domRaceLines" :key="i"
              class="flex items-start gap-2 mb-1 rounded px-1 py-0.5 transition-all duration-300"
              :class="[line.color, s >= 11 + i ? 'opacity-100' : 'opacity-0']"
            >
              <span class="font-black text-[10px] w-14 shrink-0 mt-0.5">{{ line.thread }}</span>
              <code class="text-xs leading-tight">{{ line.code }}</code>
              <span class="ml-auto text-[10px] font-black px-1 rounded shrink-0"
                :class="line.label === 'CRASH' ? 'bg-red-500 text-white' : 'bg-slate-200 text-slate-700'"
              >{{ line.label }}</span>
            </div>
          </div>
        </div>
      </div>

      <!-- Right column -->
      <div class="flex flex-col gap-2">
        <!-- The JS Solution -->
        <div :class="s >= 16 ? 'opacity-100' : 'opacity-0'" style="transition: opacity 0.4s">
          <div class="text-xs font-black text-emerald-700 uppercase tracking-wider mb-1">✅ The JavaScript Solution</div>
          <div class="flex flex-col gap-1.5">
            <div
              v-for="(item, i) in solution" :key="i"
              class="flex items-start gap-2 bg-emerald-50 border-2 border-emerald-300 rounded-lg px-3 py-1.5 text-xs font-medium transition-all duration-400"
              :class="s >= 16 + i ? 'opacity-100 translate-x-0' : 'opacity-0 translate-x-4'"
              style="transition: opacity 0.35s, transform 0.35s"
            >
              <span class="text-emerald-600 font-black mt-0.5">✓</span>
              <span class="text-emerald-900">{{ item.step }}</span>
            </div>
          </div>
        </div>

        <!-- V8 + Host Environment -->
        <div class="mt-2" :class="s >= 20 ? 'opacity-100' : 'opacity-0'" style="transition: opacity 0.5s">
          <div class="text-xs font-black text-slate-700 uppercase tracking-wider mb-1">🏗️ The Runtime Stack</div>
          <div class="flex flex-col gap-1.5">
            <div
              v-for="(e, i) in env" :key="i"
              class="flex items-center gap-2 border-2 rounded-lg px-3 py-1.5 transition-all duration-400"
              :class="[e.color, s >= 20 + i ? 'opacity-100 scale-100' : 'opacity-0 scale-95']"
              style="transition: opacity 0.35s, transform 0.35s"
            >
              <span class="text-xl">{{ e.emoji }}</span>
              <div>
                <div class="text-xs font-black">{{ e.name }}</div>
                <div class="text-[10px] opacity-80 whitespace-pre-line">{{ e.detail }}</div>
              </div>
            </div>
          </div>
        </div>

        <!-- Key insight callout -->
        <div
          class="mt-auto bg-amber-50 border-2 border-amber-400 rounded-xl px-3 py-2 transition-all duration-500"
          :class="s >= 22 ? 'opacity-100' : 'opacity-0'"
        >
          <div class="text-xs font-black text-amber-800">💡 Key Insight</div>
          <div class="text-xs text-amber-900 mt-1">JS is single-threaded, but the <strong>environment</strong> (browser/Node.js) is <strong>not</strong>. Host APIs run on OS threads, then hand results back to JS safely.</div>
        </div>
      </div>
    </div>
  </div>
</template>
