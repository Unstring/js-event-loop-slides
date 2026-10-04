<script setup lang="ts">
import { computed } from 'vue'

const props = defineProps<{ step?: number }>()
const s = computed(() => Math.min(Math.max(props.step ?? 0, 0), 22))

interface QuizItem {
  id: number
  revealStep: number
  ansStep: number
  question: string
  pillar: string
  icon: string
  options: { key: string; text: string; correct: boolean }[]
  explanation: string
}

const questions: QuizItem[] = [
  {
    id: 1,
    revealStep: 1,
    ansStep: 3,
    pillar: 'Outcome 1: Single-Threaded JS',
    icon: '🧵',
    question: 'Why did Brendan Eich make JavaScript single-threaded with an event loop?',
    options: [
      { key: 'A', text: 'CPUs in 1995 could only run one thread', correct: false },
      { key: 'B', text: 'To avoid DOM race conditions & complex locks/mutexes', correct: true },
      { key: 'C', text: 'Because single-threaded code is always faster than C++', correct: false },
    ],
    explanation: 'Single-threaded execution guarantees safe DOM manipulation without two threads trying to delete and modify the same element simultaneously.',
  },
  {
    id: 2,
    revealStep: 5,
    ansStep: 7,
    pillar: 'Outcome 2: Call Stack & Event Loop',
    icon: '⚡',
    question: 'When does the Event Loop check and dispatch pending tasks?',
    options: [
      { key: 'A', text: 'Every 5 milliseconds on a timer interrupt', correct: false },
      { key: 'B', text: 'Only when the Call Stack is completely EMPTY', correct: true },
      { key: 'C', text: 'Whenever a background network request completes', correct: false },
    ],
    explanation: 'JavaScript has run-to-completion semantics. The Event Loop can NEVER interrupt code running on the Call Stack; it only acts when the stack is empty.',
  },
  {
    id: 3,
    revealStep: 9,
    ansStep: 11,
    pillar: 'Outcome 3: setTimeout Internals',
    icon: '⏱️',
    question: 'Why does setTimeout(fn, 0) often execute after 100ms or 1000ms?',
    options: [
      { key: 'A', text: 'The delay is a MINIMUM wait before entering the queue, not execution time', correct: true },
      { key: 'B', text: 'Browser JavaScript has a 100ms random jitter bug', correct: false },
      { key: 'C', text: 'Timers only run when the user moves the mouse cursor', correct: false },
    ],
    explanation: 'The delay specifies when the Host moves the callback into the Macrotask Queue. The callback must still wait for busy stack code and all microtasks!',
  },
  {
    id: 4,
    revealStep: 13,
    ansStep: 15,
    pillar: 'Outcome 4: Microtasks vs Macrotasks',
    icon: '👑',
    question: 'What happens if a Microtask continuously queues another Microtask?',
    options: [
      { key: 'A', text: 'The browser drops old microtasks to prevent lag', correct: false },
      { key: 'B', text: 'Event loop runs one microtask, then renders, then continues', correct: false },
      { key: 'C', text: 'Microtask starvation: tab freezes completely, never renders!', correct: true },
    ],
    explanation: 'The Microtask Queue is drained to 0 to complete exhaustion. Infinite microtasks will permanently lock the thread and prevent all 60fps renders & macrotasks.',
  },
  {
    id: 5,
    revealStep: 17,
    ansStep: 19,
    pillar: 'Outcome 5: Promises & async/await',
    icon: '🤝',
    question: 'What actually happens at the "await" keyword in an async function?',
    options: [
      { key: 'A', text: 'The thread sleeps while waiting for the promise', correct: false },
      { key: 'B', text: 'Function context is saved to Heap, returns Promise, resumes via microtask', correct: true },
      { key: 'C', text: 'It creates a Web Worker thread to run the remainder in background', correct: false },
    ],
    explanation: 'await suspends the coroutine, pops its frame, returns a Promise to the caller immediately, and schedules continuation via the VIP Microtask Queue.',
  },
]
</script>

<template>
  <div class="h-full flex flex-col gap-1.5 select-none text-slate-800">
    <!-- Header -->
    <div class="flex items-center gap-3 pb-1 border-b-2 border-slate-200">
      <span class="text-2xl">🏆</span>
      <div>
        <h2 class="text-xl font-black text-slate-900 leading-tight">Mastery Knowledge Check</h2>
        <p class="text-xs text-slate-500">Teaching Demo Round · Core Conceptual Benchmark</p>
      </div>
      <div class="ml-auto flex items-center gap-2">
        <span class="px-2 py-0.5 rounded bg-emerald-100 border border-emerald-300 text-emerald-900 font-bold text-xs">
          Score: {{ Math.min(Math.floor(s / 4), 5) }} / 5 Passed
        </span>
        <div class="px-2 py-1 rounded-lg bg-sky-600 text-white text-xs font-bold">Step {{ s }}/22</div>
      </div>
    </div>

    <!-- Questions Grid -->
    <div class="grid grid-cols-5 gap-2 flex-1 min-h-0">
      <div
        v-for="q in questions" :key="q.id"
        class="border-2 rounded-xl p-2 flex flex-col justify-between transition-all duration-300"
        :class="[
          s >= q.revealStep
            ? 'opacity-100 scale-100 bg-white border-slate-300 shadow-sm'
            : 'opacity-20 scale-95 bg-slate-50 border-slate-200'
        ]"
      >
        <div>
          <div class="flex items-center justify-between mb-1">
            <span class="text-base">{{ q.icon }}</span>
            <span class="text-[8px] font-bold text-slate-500">Q{{ q.id }}</span>
          </div>
          <div class="text-[9px] font-black text-sky-800 uppercase tracking-tight mb-1">{{ q.pillar }}</div>
          <div class="text-[10px] font-bold text-slate-900 leading-tight mb-2">{{ q.question }}</div>

          <!-- Options -->
          <div class="flex flex-col gap-1">
            <div
              v-for="opt in q.options" :key="opt.key"
              class="border rounded px-1.5 py-1 text-[9px] flex items-start gap-1 transition-all duration-200"
              :class="{
                'bg-emerald-100 border-emerald-400 text-emerald-950 font-bold shadow-xs': s >= q.ansStep && opt.correct,
                'bg-rose-50 border-rose-200 text-rose-800 opacity-50': s >= q.ansStep && !opt.correct,
                'bg-slate-50 border-slate-200 text-slate-700': s < q.ansStep,
              }"
            >
              <span class="font-bold shrink-0">{{ opt.key }}.</span>
              <span class="leading-tight">{{ opt.text }}</span>
            </div>
          </div>
        </div>

        <!-- Explanation box -->
        <div
          class="mt-2 p-1.5 rounded bg-emerald-50 border border-emerald-200 text-[8px] text-emerald-900 transition-all duration-300"
          :class="s >= q.ansStep ? 'opacity-100' : 'opacity-0'"
        >
          <span class="font-bold">✓ Why:</span> {{ q.explanation }}
        </div>
      </div>
    </div>

    <!-- Bottom: Certificate / Trophy Banner -->
    <div
      class="border-2 border-emerald-400 rounded-xl p-2.5 bg-gradient-to-r from-emerald-50 via-teal-50 to-sky-50 text-slate-800 flex items-center justify-between transition-all duration-500"
      :class="s >= 21 ? 'opacity-100 translate-y-0' : 'opacity-0 translate-y-2'"
    >
      <div class="flex items-center gap-2.5">
        <span class="text-3xl">🎓</span>
        <div>
          <div class="text-xs font-black text-emerald-950">Mastery Certified: JavaScript Concurrency & Event Loop</div>
          <div class="text-[10px] text-slate-600">You now possess a complete internal mental model of V8, libuv, Web APIs, Call Stack, Microtasks & Coroutines.</div>
        </div>
      </div>
      <div class="px-3 py-1 bg-emerald-600 text-white rounded-lg text-xs font-black shrink-0 shadow-md">
        100% COMPLETE
      </div>
    </div>
  </div>
</template>
