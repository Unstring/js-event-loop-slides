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

const codeComparisonNote = computed(() => {
  if (s.value === 18) return '▶ Step 18 [Invocation]: Both fn() versions start synchronously on the Call Stack and invoke req().'
  if (s.value === 19) return '▶ Step 19 [Suspend vs Reaction]: await pops & saves context to Heap; .then() registers reaction callback.'
  if (s.value === 20) return '▶ Step 20 [Non-blocking]: Both immediately return a pending Promise to caller — thread never blocks!'
  if (s.value === 21) return '▶ Step 21 [Microtask Resume]: req() settles → Microtask queues! await coroutine resumes / .then() fires.'
  if (s.value >= 22) return '▶ Step 22 [Equivalence]: Identical bytecode execution! async/await is syntactic sugar over Promises.'
  return 'Compare what you write vs how V8 compiles and executes it under the hood.'
})
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
      <!-- LEFT: Promise Anatomy (Steps 1-12) + Animated Coroutine Engine (Steps 17-22) -->
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
        <div class="flex flex-col gap-0.5" v-if="s < 17">
          <div
            v-for="m in mechanics" :key="m.id"
            class="border rounded px-1.5 py-0.5 text-[8px] bg-white border-slate-200 flex items-center justify-between transition-all duration-200"
            :class="s >= m.id ? 'opacity-100' : 'opacity-20'"
          >
            <span class="font-bold text-slate-800">{{ m.title }}:</span>
            <span class="text-slate-600 truncate ml-1">{{ m.text }}</span>
          </div>
        </div>

        <!-- Live Coroutine Pipeline Animation (Steps 17-22 in the lower area) -->
        <div
          v-else
          class="border-2 border-violet-300 rounded-lg p-1.5 bg-violet-50/70 transition-all duration-400 flex flex-col justify-between shrink-0"
        >
          <div class="flex items-center justify-between text-[9px] font-black text-violet-900 mb-1">
            <span class="flex items-center gap-1">
              <span class="animate-spin text-xs">⚙️</span>
              <span>V8 Coroutine State Machine (Live Animation)</span>
            </span>
            <span class="bg-violet-200 text-violet-900 px-1 rounded font-mono text-[8px] font-bold">Steps 17-22</span>
          </div>

          <!-- 3-Station Animation Grid -->
          <div class="grid grid-cols-3 gap-1 text-[8px] font-mono">
            <!-- Station 1: Call Stack -->
            <div class="border rounded p-1 flex flex-col justify-between transition-all duration-300"
              :class="{
                'bg-amber-100 border-amber-400 text-amber-950 font-bold scale-[1.02] shadow-xs': s === 18,
                'bg-emerald-100 border-emerald-400 text-emerald-950 font-bold scale-[1.02] shadow-xs': s === 21,
                'bg-white border-slate-200 text-slate-600': s !== 18 && s !== 21,
              }"
            >
              <div class="font-sans font-bold text-[7.5px] uppercase text-slate-500">1. Call Stack</div>
              <div class="py-0.5 truncate text-[8.5px]">
                <span v-if="s === 17">Empty</span>
                <span v-else-if="s === 18" class="text-amber-900">⚡ fn() &gt; req()</span>
                <span v-else-if="s === 19" class="text-slate-400 italic">POPPED (Free)</span>
                <span v-else-if="s === 20" class="text-sky-700">✓ Caller runs</span>
                <span v-else-if="s === 21" class="text-emerald-800">⚡ fn() RESUMED</span>
                <span v-else class="text-green-800">Popped ✓</span>
              </div>
              <div class="text-[7px] text-slate-400 font-sans">
                {{ s === 19 ? 'Yielded Thread' : (s === 21 ? 'Restored Frame' : 'Stack Node') }}
              </div>
            </div>

            <!-- Station 2: Heap Storage -->
            <div class="border rounded p-1 flex flex-col justify-between transition-all duration-300"
              :class="{
                'bg-purple-100 border-purple-400 text-purple-950 font-bold scale-[1.02] shadow-xs ring-1 ring-purple-300': s === 19 || s === 20,
                'bg-white border-slate-200 text-slate-600': s < 19 || s >= 21,
              }"
            >
              <div class="font-sans font-bold text-[7.5px] uppercase text-slate-500">2. Heap Memory</div>
              <div class="py-0.5 truncate text-[8.5px]">
                <span v-if="s < 19">Empty</span>
                <span v-else-if="s === 19 || s === 20" class="text-purple-900 animate-pulse">📦 [[SavedCtx]]</span>
                <span v-else class="text-slate-400">Reclaimed ✓</span>
              </div>
              <div class="text-[7px] text-slate-400 font-sans">
                {{ s === 19 || s === 20 ? 'Coroutine Suspended' : 'GC Cleaned' }}
              </div>
            </div>

            <!-- Station 3: VIP Microtask Queue -->
            <div class="border rounded p-1 flex flex-col justify-between transition-all duration-300"
              :class="{
                'bg-violet-600 text-white font-bold scale-[1.02] shadow-xs animate-pulse': s === 21,
                'bg-white border-slate-200 text-slate-600': s !== 21,
              }"
            >
              <div class="font-sans font-bold text-[7.5px] uppercase" :class="s === 21 ? 'text-violet-200' : 'text-slate-500'">3. Microtasks</div>
              <div class="py-0.5 truncate text-[8.5px]">
                <span v-if="s < 21">Waiting</span>
                <span v-else-if="s === 21" class="text-white">👑 Resume Task!</span>
                <span v-else class="text-emerald-700">Drained ✓</span>
              </div>
              <div class="text-[7px] font-sans" :class="s === 21 ? 'text-violet-100' : 'text-slate-400'">
                {{ s === 21 ? 'VIP Priority Push' : 'Queue Empty' }}
              </div>
            </div>
          </div>
        </div>
      </div>

      <!-- RIGHT: async/await Compiler Magic (Steps 13-22) -->
      <div class="flex flex-col gap-1.5 justify-between min-h-0">
        <div>
          <div class="text-[10px] font-black text-slate-700 uppercase tracking-wider mb-1 flex justify-between">
            <span>⚙️ Compiler Desugaring & Execution Sync</span>
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

          <!-- Interactive Desugaring Comparison (Steps 17-22) -->
          <div
            class="border border-slate-300 rounded p-1.5 bg-slate-900 text-slate-100 transition-all duration-300"
            :class="s >= 17 ? 'opacity-100' : 'opacity-20'"
          >
            <div class="flex items-center justify-between text-[8px] font-mono mb-1">
              <span class="text-sky-400 font-bold">// Side-by-Side Runtime Trace</span>
              <span v-if="s >= 18" class="text-amber-300 bg-amber-950/80 px-1 rounded border border-amber-500/50">
                Phase: Step {{ s }}
              </span>
            </div>

            <div class="grid grid-cols-2 gap-1.5 font-mono text-[8.5px]">
              <!-- Left: async/await with live step highlights -->
              <div class="bg-slate-800 p-1.5 rounded border border-slate-700 flex flex-col justify-between">
                <div>
                  <div class="text-slate-400 text-[7.5px] uppercase flex justify-between mb-0.5">
                    <span class="text-violet-300 font-bold">1. async / await</span>
                    <span v-if="s === 18" class="text-amber-400 font-bold">Call req()</span>
                    <span v-else-if="s === 19" class="text-purple-400 font-bold animate-pulse">Suspend Heap</span>
                    <span v-else-if="s === 20" class="text-sky-400 font-bold">Pending</span>
                    <span v-else-if="s === 21" class="text-emerald-400 font-bold">Microtask Resume</span>
                    <span v-else-if="s >= 22" class="text-green-400 font-bold">Resolved ✓</span>
                  </div>
                  <div class="text-violet-300">async function fn() {</div>
                  <div
                    class="pl-2 rounded px-1 transition-colors duration-200"
                    :class="{
                      'bg-amber-500/30 text-amber-200 font-bold border-l-2 border-amber-400': s === 18,
                      'bg-purple-500/30 text-purple-200 font-bold border-l-2 border-purple-400': s === 19,
                      'bg-sky-500/20 text-sky-200': s === 20,
                      'text-slate-300': s < 18 || s > 20
                    }"
                  >
                    const x = <span class="text-amber-300 font-bold">await</span> req();
                  </div>
                  <div
                    class="pl-2 rounded px-1 transition-colors duration-200"
                    :class="{
                      'bg-emerald-500/30 text-emerald-200 font-bold border-l-2 border-emerald-400 animate-pulse': s >= 21,
                      'text-slate-300': s < 21
                    }"
                  >
                    return x;
                  </div>
                  <div class="text-violet-300">}</div>
                </div>

                <!-- Status pill -->
                <div class="mt-1 pt-1 border-t border-slate-700 text-[7.5px] flex items-center justify-between">
                  <span class="text-slate-400">Coroutine:</span>
                  <span v-if="s < 18" class="text-slate-500">Awaiting call</span>
                  <span v-else-if="s === 18" class="text-amber-300 font-bold">Executing Sync</span>
                  <span v-else-if="s === 19" class="text-purple-300 font-bold">Context in Heap</span>
                  <span v-else-if="s === 20" class="text-sky-300 font-bold">Waiting for Settlement</span>
                  <span v-else-if="s === 21" class="text-emerald-300 font-bold">Restoring from Heap</span>
                  <span v-else class="text-emerald-400 font-bold">Completed</span>
                </div>
              </div>

              <!-- Right: Promise Desugar with live step highlights -->
              <div class="bg-slate-800 p-1.5 rounded border border-slate-700 flex flex-col justify-between">
                <div>
                  <div class="text-slate-400 text-[7.5px] uppercase flex justify-between mb-0.5">
                    <span class="text-sky-300 font-bold">2. Promise Equivalent</span>
                    <span v-if="s === 18" class="text-amber-400 font-bold">Call req()</span>
                    <span v-else-if="s === 19" class="text-purple-400 font-bold animate-pulse">Reaction Record</span>
                    <span v-else-if="s === 20" class="text-sky-400 font-bold">Pending</span>
                    <span v-else-if="s === 21" class="text-emerald-400 font-bold">Microtask Drain</span>
                    <span v-else-if="s >= 22" class="text-green-400 font-bold">Resolved ✓</span>
                  </div>
                  <div class="text-sky-300">function fn() {</div>
                  <div
                    class="pl-2 rounded px-1 transition-colors duration-200"
                    :class="{
                      'bg-amber-500/30 text-amber-200 font-bold border-l-2 border-amber-400': s === 18,
                      'text-slate-300': s !== 18
                    }"
                  >
                    return Promise.resolve(req())
                  </div>
                  <div
                    class="pl-4 rounded px-1 transition-colors duration-200"
                    :class="{
                      'bg-purple-500/30 text-purple-200 font-bold border-l-2 border-purple-400': s === 19,
                      'bg-sky-500/20 text-sky-200': s === 20,
                      'bg-emerald-500/30 text-emerald-200 font-bold border-l-2 border-emerald-400 animate-pulse': s >= 21,
                      'text-emerald-300': s < 19
                    }"
                  >
                    .then(x => x);
                  </div>
                  <div class="text-sky-300">}</div>
                </div>

                <!-- Status pill -->
                <div class="mt-1 pt-1 border-t border-slate-700 text-[7.5px] flex items-center justify-between">
                  <span class="text-slate-400">Queue Status:</span>
                  <span v-if="s < 18" class="text-slate-500">Awaiting call</span>
                  <span v-else-if="s === 18" class="text-amber-300 font-bold">Executing Sync</span>
                  <span v-else-if="s === 19" class="text-purple-300 font-bold">Reaction Registered</span>
                  <span v-else-if="s === 20" class="text-sky-300 font-bold">Waiting for Settlement</span>
                  <span v-else-if="s === 21" class="text-emerald-300 font-bold">VIP Microtask Run</span>
                  <span v-else class="text-emerald-400 font-bold">Completed</span>
                </div>
              </div>
            </div>
          </div>
        </div>

        <!-- Dynamic Step Commentary & Key Rule Banner -->
        <div
          class="border rounded p-1.5 transition-all duration-300 shrink-0 text-[8.5px]"
          :class="s >= 18 ? 'bg-amber-50 border-amber-300 text-amber-950 font-medium' : 'bg-emerald-50 border-emerald-300 text-emerald-950'"
        >
          <div class="flex items-center gap-1">
            <span class="font-bold shrink-0">💡 Execution State:</span>
            <span class="leading-tight">{{ codeComparisonNote }}</span>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
