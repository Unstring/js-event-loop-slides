<script setup lang="ts">
import { computed } from 'vue'

const props = defineProps<{ step?: number }>()
const s = computed(() => Math.min(Math.max(props.step ?? 0, 0), 22))

const promiseStates = [
  { id: 2, name: 'Pending', desc: 'Initial state. Unsettled.', color: 'bg-amber-100 border-amber-400 text-amber-900', icon: '⏳' },
  { id: 3, name: 'Fulfilled', desc: 'Success. [[PromiseResult]] is frozen value.', color: 'bg-emerald-100 border-emerald-400 text-emerald-900', icon: '✅' },
  { id: 4, name: 'Rejected', desc: 'Failure. [[PromiseResult]] is error reason.', color: 'bg-rose-100 border-rose-400 text-rose-900', icon: '❌' },
]

const mechanics = [
  { id: 7, title: 'Reaction Records', text: '.then() registers a reaction in [[PromiseFulfillReactions]].' },
  { id: 8, title: 'Microtask Trigger', text: 'resolve() moves reactions to the VIP Microtask Queue.' },
  { id: 9, title: 'Always Async', text: '.then() callback is ALWAYS deferred to microtasks, even if already resolved!' },
]

const asyncAwaitPoints = [
  { id: 13, title: '1. Sync until await', text: 'Executes immediately on main stack until first await.' },
  { id: 14, title: '2. Stack Unwind', text: 'At await, V8 suspends context & saves variables to Heap.' },
  { id: 15, title: '3. Caller Continues', text: 'Async function returns a pending Promise immediately.' },
  { id: 16, title: '4. Microtask Resume', text: 'When awaited Promise settles, microtask resumes function.' },
]
</script>

<template>
  <div class="h-full flex flex-col justify-between select-none text-slate-800 text-xs">
    <!-- Header -->
    <div class="flex items-center gap-2 pb-1 border-b border-slate-200 shrink-0">
      <span class="text-xl">🤝</span>
      <div>
        <h2 class="text-base font-black text-slate-900 leading-tight">Promises & async/await Internals</h2>
        <p class="text-[10px] text-slate-500">Chapter 6 of 6 · State Machine & Suspend/Resume Architecture</p>
      </div>
      <div class="ml-auto flex items-center gap-1.5">
        <span class="px-2 py-0.5 rounded bg-sky-100 text-sky-900 text-[10px] font-bold">V8 Coroutine Machine</span>
        <div class="px-2 py-0.5 rounded bg-sky-600 text-white text-[10px] font-bold">Step {{ s }}/22</div>
      </div>
    </div>

    <!-- 2 Columns -->
    <div class="grid grid-cols-2 gap-2 flex-1 min-h-0 py-1">
      <!-- LEFT: Promise Anatomy (Steps 1-12) -->
      <div class="flex flex-col gap-1.5 justify-between min-h-0">
        <div>
          <div class="text-[10px] font-black text-slate-700 uppercase tracking-wider mb-1 flex justify-between">
            <span>🔮 Anatomy of a Promise (V8 Internal Slot)</span>
            <span class="text-[9px] text-slate-400 font-mono">Steps 1-12</span>
          </div>

          <!-- 3 States -->
          <div class="grid grid-cols-3 gap-1 mb-1">
            <div
              v-for="st in promiseStates" :key="st.id"
              class="border rounded p-1 flex flex-col transition-all duration-200"
              :class="[st.color, s >= st.id ? 'opacity-100' : 'opacity-20']"
            >
              <div class="flex items-center gap-1 mb-0.5">
                <span class="text-xs">{{ st.icon }}</span>
                <span class="text-[8px] font-black uppercase">{{ st.name }}</span>
              </div>
              <div class="text-[7px] leading-tight opacity-90">{{ st.desc }}</div>
            </div>
          </div>

          <!-- Internal Slots Box -->
          <div
            class="border border-slate-300 rounded p-1.5 bg-slate-900 text-slate-200 font-mono text-[8px] transition-all duration-300 flex flex-col gap-0.5"
            :class="s >= 5 ? 'opacity-100' : 'opacity-20'"
          >
            <div class="text-amber-400 font-bold">// JSPromise Internal Representation</div>
            <div>[[PromiseState]]: <span class="text-emerald-400">"fulfilled"</span></div>
            <div>[[PromiseResult]]: <span class="text-sky-300">42</span> <span class="text-slate-500">// Immutable</span></div>
            <div>[[PromiseFulfillReactions]]: <span class="text-violet-300">[ { handler, promiseOrCapability } ]</span></div>
          </div>
        </div>

        <!-- Reaction Records (Steps 7-12) -->
        <div class="flex flex-col gap-0.5">
          <div
            v-for="m in mechanics" :key="m.id"
            class="border rounded px-1.5 py-0.5 text-[8px] bg-white border-slate-200 flex items-center justify-between transition-all duration-200"
            :class="s >= m.id ? 'opacity-100' : 'opacity-20'"
          >
            <span class="font-bold text-slate-800">{{ m.title }}:</span>
            <span class="text-slate-600 truncate ml-1">{{ m.text }}</span>
          </div>
        </div>
      </div>

      <!-- RIGHT: async/await Compiler Magic (Steps 13-22) -->
      <div class="flex flex-col gap-1.5 justify-between min-h-0">
        <div>
          <div class="text-[10px] font-black text-slate-700 uppercase tracking-wider mb-1 flex justify-between">
            <span>⚙️ How async / await Works</span>
            <span class="text-[9px] text-violet-700 font-mono font-bold">Steps 13-22</span>
          </div>

          <!-- 4 steps grid -->
          <div class="grid grid-cols-2 gap-1 mb-1">
            <div
              v-for="step in asyncAwaitPoints" :key="step.id"
              class="border rounded p-1 flex flex-col transition-all duration-200"
              :class="s >= step.id ? 'bg-violet-50/70 border-violet-300 text-violet-950 opacity-100' : 'border-transparent text-slate-300 opacity-20'"
            >
              <div class="font-bold text-[8px] text-violet-900">{{ step.title }}</div>
              <div class="text-[7px] text-slate-600 leading-tight">{{ step.text }}</div>
            </div>
          </div>

          <!-- Desugaring Comparison -->
          <div
            class="border border-slate-300 rounded p-1.5 bg-slate-900 text-slate-100 transition-all duration-300"
            :class="s >= 17 ? 'opacity-100' : 'opacity-20'"
          >
            <div class="text-[8px] font-bold text-sky-400 font-mono mb-0.5">// What you write vs V8 compilation:</div>
            <div class="grid grid-cols-2 gap-1 font-mono text-[8px]">
              <div class="bg-slate-800 p-1 rounded border border-slate-700 text-slate-300">
                <div class="text-violet-300">async function fn() {</div>
                <div class="pl-2">const x = <span class="text-amber-300">await</span> req();</div>
                <div class="pl-2">return x;</div>
                <div class="text-violet-300">}</div>
              </div>
              <div class="bg-slate-800 p-1 rounded border border-slate-700 text-slate-300">
                <div class="text-sky-300">function fn() {</div>
                <div class="pl-2">return Promise.resolve(req())</div>
                <div class="pl-4 text-emerald-300">.then(x => x);</div>
                <div class="text-sky-300">}</div>
              </div>
            </div>
          </div>
        </div>

        <!-- Key Takeaway Banner -->
        <div class="border border-emerald-300 rounded p-1 bg-emerald-50 text-emerald-950 shrink-0 text-[8px]">
          <strong>Key Rule:</strong> <code class="bg-white px-1 rounded border border-emerald-200">await</code> does NOT block the thread! It suspends the coroutine to Heap and returns a Promise to the caller immediately.
        </div>
      </div>
    </div>
  </div>
</template>
