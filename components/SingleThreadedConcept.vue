<script setup lang="ts">
import { computed } from 'vue'

const props = defineProps<{ step?: number }>()
const s = computed(() => Math.min(Math.max(props.step ?? 0, 0), 22))

const threadBasics = [
  { id: 1, text: 'A thread is a single sequential stream of CPU instructions.', tag: 'Definition' },
  { id: 2, text: 'Multi-threaded engines (Java/C++) run instructions on multiple cores simultaneously.', tag: 'Multi-Thread' },
  { id: 3, text: 'JavaScript is single-threaded: exactly ONE Call Stack executing at any millisecond.', tag: 'JS Core' },
  { id: 4, text: 'One thread means no race conditions, no locks, and no thread deadlocks.', tag: 'Benefit' },
  { id: 5, text: 'Downside: If code blocks the main thread, the entire browser window freezes!', tag: 'Tradeoff' },
]

const raceSteps = [
  { id: 6, who: 'Thread A', action: 'const box = document.getElementById("box");', status: 'Reads DOM', color: 'text-sky-700 bg-sky-50 border-sky-300' },
  { id: 7, who: 'Thread B', action: 'box.parentNode.removeChild(box);', status: 'Deletes DOM node', color: 'text-rose-700 bg-rose-50 border-rose-300' },
  { id: 8, who: 'Thread A', action: 'box.style.backgroundColor = "red";', status: 'Writes to deleted node', color: 'text-amber-700 bg-amber-50 border-amber-300' },
  { id: 9, who: 'SYSTEM', action: 'CRASH: NullPointerDereference / Memory Corruption', status: '💥 SEGFAULT', color: 'text-white bg-red-600 border-red-700 font-bold' },
  { id: 10, who: 'Why JS', action: 'Brendan Eich in 1995: Avoid mutexes, deadlocks & race conditions in UI.', status: 'Decision', color: 'text-emerald-800 bg-emerald-50 border-emerald-300' },
]

const solutionPillars = [
  { id: 11, title: '1. Single Mutator', desc: 'Only one thread ever manipulates the DOM tree.' },
  { id: 12, title: '2. Run-to-Completion', desc: 'A running function is NEVER interrupted mid-execution.' },
  { id: 13, title: '3. Host Offloading', desc: 'Timers, disk I/O, & network requests run on OS threads.' },
  { id: 14, title: '4. Non-Blocking Callbacks', desc: 'Host puts finished tasks into queues for the main thread.' },
]

const runtimeLayers = [
  { id: 16, name: 'V8 Engine', desc: 'Call Stack + Memory Heap (Single-Threaded)', icon: '⚡', color: 'bg-sky-50 border-sky-300 text-sky-950' },
  { id: 17, name: 'Browser Host APIs', desc: 'C++ thread pool for DOM, Timers, HTTP, GPU', icon: '🌐', color: 'bg-emerald-50 border-emerald-300 text-emerald-950' },
  { id: 18, name: 'Node.js / libuv', desc: 'Event demultiplexer + 4-thread worker pool', icon: '⚙️', color: 'bg-violet-50 border-violet-300 text-violet-950' },
  { id: 19, name: 'The Event Loop', desc: 'Coordinates between Call Stack & Host queues', icon: '↻', color: 'bg-amber-50 border-amber-300 text-amber-950' },
]
</script>

<template>
  <div class="h-full flex flex-col justify-between select-none text-slate-800 text-xs">
    <!-- Header -->
    <div class="flex items-center gap-2 pb-1 border-b border-slate-200 shrink-0">
      <span class="text-xl">🧵</span>
      <div>
        <h2 class="text-base font-black text-slate-900 leading-tight">Why is JavaScript Single-Threaded?</h2>
        <p class="text-[10px] text-slate-500">Chapter 1 of 5 · Architectural Foundations</p>
      </div>
      <div class="ml-auto flex items-center gap-1.5">
        <span class="px-2 py-0.5 rounded bg-sky-100 text-sky-800 text-[10px] font-bold">1 Stack · 1 Thread</span>
        <div class="px-2 py-0.5 rounded bg-sky-600 text-white text-[10px] font-bold">Step {{ s }}/22</div>
      </div>
    </div>

    <!-- Main 2-Column Content -->
    <div class="grid grid-cols-2 gap-2 flex-1 min-h-0 py-1">
      <!-- LEFT COLUMN -->
      <div class="flex flex-col gap-1.5 justify-between min-h-0">
        <!-- Part 1: What is a thread (Steps 1-5) -->
        <div class="border border-slate-200 rounded-lg p-2 bg-slate-50/70 flex flex-col gap-1">
          <div class="text-[10px] font-black text-slate-700 uppercase tracking-wider flex justify-between">
            <span>📌 What is a Thread?</span>
            <span class="text-[9px] text-slate-400 font-mono">Steps 1-5</span>
          </div>
          <div class="flex flex-col gap-0.5">
            <div
              v-for="item in threadBasics" :key="item.id"
              class="px-1.5 py-0.5 rounded border text-[10px] flex items-center justify-between transition-all duration-200"
              :class="[
                s >= item.id ? 'opacity-100 bg-white border-slate-300 text-slate-900' : 'opacity-20 bg-transparent border-transparent text-slate-400'
              ]"
            >
              <span class="truncate pr-1"><strong>{{ item.id }}.</strong> {{ item.text }}</span>
              <span class="text-[8px] px-1 py-0.2 rounded bg-slate-100 text-slate-600 shrink-0 font-bold">{{ item.tag }}</span>
            </div>
          </div>
        </div>

        <!-- Part 2: DOM Race Condition (Steps 6-10) -->
        <div
          class="border border-rose-200 rounded-lg p-2 bg-rose-50/40 flex flex-col gap-1 transition-all duration-300"
          :class="s >= 6 ? 'opacity-100' : 'opacity-25'"
        >
          <div class="text-[10px] font-black text-rose-800 uppercase tracking-wider flex justify-between">
            <span>💥 The Multi-Threaded DOM Disaster</span>
            <span class="text-[9px] text-rose-500 font-mono">Steps 6-10</span>
          </div>
          <div class="flex flex-col gap-1">
            <div
              v-for="race in raceSteps" :key="race.id"
              class="px-2 py-0.5 rounded border text-[10px] font-mono flex items-center justify-between transition-all duration-200"
              :class="[
                s >= race.id ? race.color + ' opacity-100 scale-100' : 'opacity-15 bg-white border-slate-100 text-slate-400 scale-98'
              ]"
            >
              <div class="flex items-center gap-1.5 truncate">
                <span class="text-[9px] font-bold shrink-0 opacity-80">[{{ race.who }}]</span>
                <span class="truncate">{{ race.action }}</span>
              </div>
              <span class="text-[8px] px-1 rounded bg-black/10 shrink-0 font-sans font-bold">{{ race.status }}</span>
            </div>
          </div>
        </div>
      </div>

      <!-- RIGHT COLUMN -->
      <div class="flex flex-col gap-1.5 justify-between min-h-0">
        <!-- Part 3: The JS Solution (Steps 11-15) -->
        <div
          class="border border-emerald-200 rounded-lg p-2 bg-emerald-50/40 flex flex-col gap-1 transition-all duration-300"
          :class="s >= 11 ? 'opacity-100' : 'opacity-25'"
        >
          <div class="text-[10px] font-black text-emerald-800 uppercase tracking-wider flex justify-between">
            <span>✅ The JavaScript Solution</span>
            <span class="text-[9px] text-emerald-600 font-mono">Steps 11-15</span>
          </div>
          <div class="grid grid-cols-2 gap-1">
            <div
              v-for="item in solutionPillars" :key="item.id"
              class="p-1 rounded border text-[10px] transition-all duration-200"
              :class="s >= item.id ? 'opacity-100 bg-white border-emerald-300 text-emerald-950' : 'opacity-20 border-transparent text-slate-400'"
            >
              <div class="font-bold text-[9px] text-emerald-700">{{ item.title }}</div>
              <div class="text-[9px] text-slate-600 leading-tight">{{ item.desc }}</div>
            </div>
          </div>
        </div>

        <!-- Part 4: Runtime Architecture (Steps 16-19) -->
        <div
          class="border border-sky-200 rounded-lg p-2 bg-sky-50/40 flex flex-col gap-1 transition-all duration-300"
          :class="s >= 16 ? 'opacity-100' : 'opacity-25'"
        >
          <div class="text-[10px] font-black text-sky-800 uppercase tracking-wider flex justify-between">
            <span>🏗️ Runtime Stack: JS Engine vs Host Environment</span>
            <span class="text-[9px] text-sky-600 font-mono">Steps 16-19</span>
          </div>
          <div class="grid grid-cols-2 gap-1">
            <div
              v-for="layer in runtimeLayers" :key="layer.id"
              class="p-1 rounded border text-[10px] flex items-center gap-1.5 transition-all duration-200"
              :class="s >= layer.id ? layer.color + ' opacity-100' : 'opacity-20 bg-white border-slate-100 text-slate-400'"
            >
              <span class="text-sm shrink-0">{{ layer.icon }}</span>
              <div class="truncate">
                <div class="font-bold text-[9px] truncate">{{ layer.name }}</div>
                <div class="text-[8px] opacity-80 truncate">{{ layer.desc }}</div>
              </div>
            </div>
          </div>
        </div>

        <!-- Part 5: Grand Takeaway (Steps 20-22) -->
        <div
          class="border border-amber-300 rounded-lg p-1.5 bg-amber-50 text-amber-950 transition-all duration-300 shrink-0"
          :class="s >= 20 ? 'opacity-100 translate-y-0' : 'opacity-0 translate-y-2'"
        >
          <div class="flex items-center justify-between text-[10px] font-black mb-0.5">
            <span class="flex items-center gap-1">💡 The Golden Runtime Paradox</span>
            <span class="text-[8px] bg-amber-200 text-amber-900 px-1 rounded">Steps 20-22</span>
          </div>
          <div class="text-[9px] leading-snug">
            <span v-if="s >= 20"><strong>Paradox:</strong> How can JS be single-threaded yet handle millions of concurrent connections? </span>
            <span v-if="s >= 21" class="text-emerald-800 font-bold">Answer: </span>
            <span v-if="s >= 21">The <em>V8 execution engine</em> is single-threaded. The <em>browser / Node.js host environment</em> is heavily multi-threaded! </span>
            <span v-if="s >= 22" class="text-violet-800 font-bold">The Event Loop is the bridge between them.</span>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
