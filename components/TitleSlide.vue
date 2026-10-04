<script setup lang="ts">
import { computed } from 'vue'
const props = defineProps<{ step?: number }>()
const s = computed(() => Math.min(Math.max(props.step ?? 0, 0), 8))

const topics = [
  { emoji: '🧵', label: 'Single-Threaded Model', color: 'bg-sky-100 border-sky-400 text-sky-900' },
  { emoji: '📚', label: 'Call Stack & Event Loop', color: 'bg-violet-100 border-violet-400 text-violet-900' },
  { emoji: '⏱️', label: 'setTimeout Internals', color: 'bg-amber-100 border-amber-400 text-amber-900' },
  { emoji: '⚡', label: 'Micro vs Macro Tasks', color: 'bg-emerald-100 border-emerald-400 text-emerald-900' },
  { emoji: '🤝', label: 'Promises & async/await', color: 'bg-rose-100 border-rose-400 text-rose-900' },
]

const code = `console.log('A')
setTimeout(() => console.log('B'), 0)
Promise.resolve().then(() => console.log('C'))
console.log('D')`

const outputs = ['A', 'D', 'C', 'B']
</script>

<template>
  <div class="h-full flex flex-col justify-between select-none px-2 py-1">
    <!-- Header -->
    <div class="text-center pt-2">
      <div class="inline-flex items-center gap-2 px-3 py-1 rounded-full bg-sky-600 text-white text-xs font-bold tracking-widest uppercase mb-3">
        Teaching Demo Round · Core JavaScript
      </div>
      <h1 class="text-4xl font-black text-slate-900 leading-tight tracking-tight">
        The JavaScript<br/>
        <span class="text-sky-600">Event Loop</span>
      </h1>
      <p class="text-slate-500 text-sm mt-1">Asynchronous Concurrency & Runtime Architecture</p>
    </div>

    <!-- Topics Grid -->
    <div class="grid grid-cols-5 gap-2 mt-3">
      <div
        v-for="(t, i) in topics" :key="i"
        class="border-2 rounded-xl px-3 py-2 text-center transition-all duration-500 font-semibold text-xs flex flex-col items-center gap-1"
        :class="[t.color, s >= i + 1 ? 'opacity-100 scale-100' : 'opacity-0 scale-90']"
      >
        <span class="text-xl">{{ t.emoji }}</span>
        <span class="leading-tight">{{ t.label }}</span>
      </div>
    </div>

    <!-- Mystery Code Box -->
    <div
      class="mt-3 transition-all duration-500 rounded-xl border-2 border-slate-300 bg-slate-50 p-3 font-mono text-sm"
      :class="s >= 6 ? 'opacity-100' : 'opacity-0'"
    >
      <div class="flex items-center justify-between mb-2">
        <span class="text-xs font-bold text-slate-600 uppercase tracking-wide">🔍 Mystery — What logs first?</span>
        <span class="text-[10px] text-slate-400">Press → to reveal</span>
      </div>
      <div class="grid grid-cols-2 gap-3">
        <pre class="text-slate-800 text-xs leading-relaxed">{{ code }}</pre>
        <div class="flex flex-col justify-center gap-1">
          <div class="text-xs font-bold text-slate-600 mb-1">Output Order:</div>
          <div class="flex gap-2">
            <span
              v-for="(o, i) in outputs" :key="i"
              class="w-8 h-8 rounded-lg flex items-center justify-center font-black text-sm border-2 transition-all duration-300"
              :class="s >= 7
                ? (o === 'A' || o === 'D' ? 'bg-sky-600 text-white border-sky-600' : o === 'C' ? 'bg-violet-600 text-white border-violet-600' : 'bg-amber-500 text-white border-amber-500')
                : 'bg-white border-slate-300 text-slate-400'"
            >{{ s >= 7 ? o : '?' }}</span>
          </div>
          <div v-if="s >= 7" class="text-[10px] text-slate-600 mt-1">
            <span class="text-sky-600 font-bold">A, D</span> = Sync •
            <span class="text-violet-600 font-bold">C</span> = Microtask •
            <span class="text-amber-500 font-bold">B</span> = Macrotask
          </div>
        </div>
      </div>
    </div>

    <!-- Footer -->
    <div
      class="text-center text-xs text-slate-400 pb-1 transition-all duration-500"
      :class="s >= 8 ? 'opacity-100' : 'opacity-0'"
    >
      Press <kbd class="px-1.5 py-0.5 rounded border border-slate-300 bg-white text-slate-600 font-mono text-xs">→</kbd> to step through every concept
    </div>
  </div>
</template>
