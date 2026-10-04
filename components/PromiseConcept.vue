<script setup lang="ts">
import { computed } from 'vue'

const props = defineProps<{ step?: number }>()
const s = computed(() => Math.min(Math.max(props.step ?? 0, 0), 22))

const promiseStates = [
  { id: 2, name: 'Pending', desc: 'Initial state. Neither fulfilled nor rejected. Has observer reaction lists.', color: 'bg-amber-100 border-amber-400 text-amber-900', icon: '⏳' },
  { id: 3, name: 'Fulfilled', desc: 'Operation succeeded. Holds immutable Result [[PromiseResult]].', color: 'bg-emerald-100 border-emerald-400 text-emerald-900', icon: '✅' },
  { id: 4, name: 'Rejected', desc: 'Operation failed. Holds immutable Reason/Error [[PromiseResult]].', color: 'bg-rose-100 border-rose-400 text-rose-900', icon: '❌' },
]

const mechanics = [
  { id: 7, title: 'Reaction Records', text: '.then() does not run immediately — it registers a reaction in [[PromiseFulfillReactions]].' },
  { id: 8, title: 'Microtask Trigger', text: 'When resolve() is called, all attached reactions are enqueued as Microtasks.' },
  { id: 9, title: 'Always Asynchronous', text: 'Even if a Promise is already fulfilled, .then() callback is ALWAYS deferred to microtask queue!' },
]

const asyncAwaitPoints = [
  { id: 13, title: '1. Synchronous until await', text: 'Everything up to the first await keyword executes immediately on the main call stack.' },
  { id: 14, title: '2. Stack Unwinding & Save', text: 'At await, V8 suspends the function context, saves local variables to Heap, and yields thread.' },
  { id: 15, title: '3. Caller gets Promise', text: 'The async function returns a pending Promise to its caller immediately without blocking.' },
  { id: 16, title: '4. Microtask Resumption', text: 'When awaited Promise settles, a microtask resumes the function right after the await line.' },
]
</script>

<template>
  <div class="h-full flex flex-col gap-1.5 select-none text-slate-800">
    <!-- Header -->
    <div class="flex items-center gap-3 pb-1 border-b-2 border-slate-200">
      <span class="text-2xl">🤝</span>
      <div>
        <h2 class="text-xl font-black text-slate-900 leading-tight">Promises & async/await Internals</h2>
        <p class="text-xs text-slate-500">Chapter 6 of 6 · State Machine & Suspend/Resume</p>
      </div>
      <div class="ml-auto px-2 py-1 rounded-lg bg-sky-600 text-white text-xs font-bold">Step {{ s }}/22</div>
    </div>

    <!-- Subtitle Banner -->
    <div class="bg-slate-50 border border-slate-200 rounded-lg px-3 py-1 text-xs text-slate-700 flex items-center justify-between">
      <span>
        A <strong class="text-sky-700">Promise</strong> is an immutable state container; <strong class="text-violet-700">async/await</strong> is a compiler transformation turning asynchronous callbacks into suspendable state machines.
      </span>
      <span class="text-[10px] text-slate-500 font-mono">ECMA-262 § 27.2</span>
    </div>

    <!-- Main Grid -->
    <div class="grid grid-cols-2 gap-3 flex-1 min-h-0">
      <!-- LEFT: Promise Anatomy & States -->
      <div class="flex flex-col gap-2">
        <div class="text-xs font-black text-slate-700 uppercase tracking-wider">
          🔮 Anatomy of a Promise Object (V8 Internal Slot)
        </div>

        <!-- 3 States -->
        <div class="grid grid-cols-3 gap-2">
          <div
            v-for="st in promiseStates" :key="st.id"
            class="border-2 rounded-xl p-2 flex flex-col transition-all duration-300"
            :class="[
              st.color,
              s >= st.id ? 'opacity-100 scale-100 shadow-sm' : 'opacity-0 scale-95'
            ]"
          >
            <div class="flex items-center justify-between mb-1">
              <span class="text-base">{{ st.icon }}</span>
              <span class="text-[10px] font-black uppercase">{{ st.name }}</span>
            </div>
            <div class="text-[9px] leading-tight font-medium opacity-90">{{ st.desc }}</div>
          </div>
        </div>

        <!-- Internal slots box -->
        <div
          class="border-2 border-slate-200 rounded-xl p-2.5 bg-slate-900 text-slate-200 transition-all duration-300"
          :class="s >= 5 ? 'opacity-100' : 'opacity-0'"
        >
          <div class="text-[10px] font-black text-amber-400 font-mono mb-1">// V8 JSAsyncFunction / JSPromise Internal Representation</div>
          <div class="font-mono text-[10px] flex flex-col gap-0.5 text-slate-300">
            <div>[[PromiseState]]: <span class="text-emerald-400">"fulfilled"</span></div>
            <div>[[PromiseResult]]: <span class="text-sky-300">42</span> <span class="text-slate-500">// Frozen forever</span></div>
            <div>[[PromiseFulfillReactions]]: <span class="text-violet-300">[ { handler: fn, capability } ]</span></div>
            <div>[[PromiseRejectReactions]]: <span class="text-rose-300">[ ]</span></div>
          </div>
        </div>

        <!-- How reactions work -->
        <div class="flex flex-col gap-1 mt-1">
          <div
            v-for="m in mechanics" :key="m.id"
            class="border rounded-lg px-2 py-1.5 text-xs bg-slate-50 border-slate-200 flex flex-col transition-all duration-300"
            :class="s >= m.id ? 'opacity-100 translate-x-0' : 'opacity-0 -translate-x-3'"
          >
            <span class="font-bold text-slate-800 text-[10px]">{{ m.title }}</span>
            <span class="text-slate-600 text-[10px]">{{ m.text }}</span>
          </div>
        </div>
      </div>

      <!-- RIGHT: async/await Compiler Magic -->
      <div class="flex flex-col gap-2">
        <div class="text-xs font-black text-slate-700 uppercase tracking-wider flex items-center justify-between">
          <span>⚙️ How async / await Actually Works</span>
          <span class="text-[9px] bg-violet-100 text-violet-800 font-bold px-1.5 py-0.5 rounded">Compiler desugaring</span>
        </div>

        <!-- 4 steps of async/await -->
        <div class="flex flex-col gap-1">
          <div
            v-for="step in asyncAwaitPoints" :key="step.id"
            class="border-2 rounded-lg p-2 text-xs flex flex-col transition-all duration-300"
            :class="[
              s >= step.id ? 'opacity-100 translate-y-0 bg-violet-50/70 border-violet-300' : 'opacity-0 translate-y-2 bg-white border-slate-100'
            ]"
          >
            <div class="font-black text-violet-950 text-[11px] mb-0.5">{{ step.title }}</div>
            <div class="text-slate-600 text-[10px] leading-snug">{{ step.text }}</div>
          </div>
        </div>

        <!-- Code Desugaring Visual -->
        <div
          class="border-2 border-slate-300 rounded-xl p-2.5 bg-slate-900 text-slate-100 transition-all duration-300 flex-1 flex flex-col justify-between"
          :class="s >= 17 ? 'opacity-100' : 'opacity-0'"
        >
          <div>
            <div class="text-[10px] font-black text-sky-400 font-mono mb-1">
              // What you write vs What V8 executes:
            </div>
            <div class="grid grid-cols-2 gap-2 font-mono text-[9px]">
              <!-- Left: async/await -->
              <div class="bg-slate-800 p-2 rounded border border-slate-700">
                <div class="text-slate-400 mb-1">// You write:</div>
                <div class="text-violet-300">async function fetchUser() {</div>
                <div class="pl-2 text-slate-300">const res = <span class="text-amber-300">await</span> api();</div>
                <div class="pl-2 text-slate-300">return res.data;</div>
                <div class="text-violet-300">}</div>
              </div>

              <!-- Right: Promise desugar -->
              <div class="bg-slate-800 p-2 rounded border border-slate-700">
                <div class="text-slate-400 mb-1">// V8 creates:</div>
                <div class="text-sky-300">function fetchUser() {</div>
                <div class="pl-2 text-slate-300">return Promise.resolve(api())</div>
                <div class="pl-4 text-emerald-300">.then(res => res.data);</div>
                <div class="text-sky-300">}</div>
              </div>
            </div>
          </div>

          <div class="mt-2 text-[10px] bg-slate-800/90 p-1.5 rounded border border-amber-500/40 text-amber-200">
            💡 <strong>Key takeaway:</strong> <code class="text-amber-300">await</code> is never thread-blocking! It simply schedules the rest of your function as a <strong>Microtask</strong>!
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
