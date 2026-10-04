<script setup lang="ts">
import { computed } from 'vue'

const props = defineProps<{ step?: number }>()
const s = computed(() => Math.min(Math.max(props.step ?? 0, 0), 22))

const phases = [
  { id: 7, num: '1', label: 'Call Stack Check', detail: 'Is Call Stack depth === 0? If busy, loop waits.', color: 'bg-sky-50 border-sky-300 text-sky-950', icon: '⚡' },
  { id: 9, num: '2', label: 'Drain Microtasks', detail: 'Drain ALL Promise & queueMicrotask callbacks to 0.', color: 'bg-violet-50 border-violet-300 text-violet-950', icon: '👑' },
  { id: 11, num: '3', label: 'Render Opportunity', detail: 'V-Sync pulse (16.6ms)? Run rAF → Layout → Paint.', color: 'bg-rose-50 border-rose-300 text-rose-950', icon: '🎨' },
  { id: 13, num: '4', label: 'Run ONE Macrotask', detail: 'Dequeue exactly 1 timer / I/O callback onto stack.', color: 'bg-amber-50 border-amber-300 text-amber-950', icon: '⏱️' },
]

const envs = [
  {
    id: 15,
    name: 'Browser Host (Chrome / WebKit)',
    icon: '🌐',
    points: ['C++ Web APIs handle timers & network', 'Syncs to 60Hz/120Hz display refresh', 'UI Events (click, scroll) queue as macrotasks'],
    color: 'bg-sky-50 border-sky-300 text-sky-950'
  },
  {
    id: 18,
    name: 'Node.js Host (libuv runtime)',
    icon: '⚙️',
    points: ['libuv event demultiplexer (epoll/kqueue)', '4-thread worker pool for file I/O & crypto', 'process.nextTick has highest microtask priority'],
    color: 'bg-emerald-50 border-emerald-300 text-emerald-950'
  }
]
</script>

<template>
  <div class="h-full flex flex-col justify-between select-none text-slate-800 text-xs">
    <!-- Header -->
    <div class="flex items-center gap-2 pb-1 border-b border-slate-200 shrink-0">
      <span class="text-xl">↻</span>
      <div>
        <h2 class="text-base font-black text-slate-900 leading-tight">The Event Loop: Architectural Heartbeat</h2>
        <p class="text-[10px] text-slate-500">Chapter 3 of 5 · 4-Phase Cycle & Host Environments</p>
      </div>
      <div class="ml-auto flex items-center gap-1.5">
        <span class="px-2 py-0.5 rounded bg-amber-100 text-amber-900 text-[10px] font-bold">Infinite Coordinator</span>
        <div class="px-2 py-0.5 rounded bg-sky-600 text-white text-[10px] font-bold">Step {{ s }}/22</div>
      </div>
    </div>

    <!-- 2 Main Columns -->
    <div class="grid grid-cols-2 gap-2 flex-1 min-h-0 py-1">
      <!-- LEFT: What is it & C++ Engine Loop (Steps 1-6) + Host Environments (Steps 14-19) -->
      <div class="flex flex-col gap-1.5 justify-between min-h-0">
        <!-- Definition & C++ while loop -->
        <div class="border border-slate-200 rounded-lg p-2 bg-slate-50 flex flex-col gap-1">
          <div class="text-[10px] font-black text-slate-700 uppercase tracking-wider flex justify-between">
            <span>📌 What is the Event Loop?</span>
            <span class="text-[9px] text-slate-400 font-mono">Steps 1-6</span>
          </div>
          <div class="text-[10px] text-slate-700 leading-snug">
            The Event Loop is an <strong class="text-amber-800">infinite coordinator loop</strong> inside V8 and libuv that pumps callbacks from queues to the Call Stack.
          </div>

          <!-- C++ Pseudocode -->
          <div class="bg-slate-900 rounded p-2 font-mono text-[9px] text-slate-300 flex flex-col gap-0.5">
            <div class="text-slate-500">// V8 / WHATWG Event Loop Specification</div>
            <div :class="s >= 2 ? 'text-amber-300 font-bold' : 'text-slate-600'">while (eventLoop.isRunning()) {</div>
            <div class="pl-3" :class="s >= 3 ? 'text-sky-300' : 'text-slate-600'">if (callStack.isEmpty()) {</div>
            <div class="pl-6" :class="s >= 4 ? 'text-violet-300 font-bold' : 'text-slate-600'">drainMicrotasks(); // Run ALL</div>
            <div class="pl-6" :class="s >= 5 ? 'text-rose-300 font-bold' : 'text-slate-600'">if (isRenderTime()) renderUI();</div>
            <div class="pl-6" :class="s >= 6 ? 'text-amber-300 font-bold' : 'text-slate-600'">runOneMacrotask(); // Exactly 1</div>
            <div class="pl-3" :class="s >= 3 ? 'text-sky-300' : 'text-slate-600'">}</div>
            <div :class="s >= 2 ? 'text-amber-300 font-bold' : 'text-slate-600'">}</div>
          </div>
        </div>

        <!-- Host Environments: Browser vs Node.js (Steps 14-19) -->
        <div
          class="border border-slate-200 rounded-lg p-1.5 bg-white flex flex-col gap-1 transition-all duration-300 shrink-0"
          :class="s >= 14 ? 'opacity-100' : 'opacity-25'"
        >
          <div class="text-[10px] font-black text-slate-700 uppercase tracking-wider flex justify-between">
            <span>🌐 Host Environments (Where Timers Live)</span>
            <span class="text-[9px] text-slate-400 font-mono">Steps 14-19</span>
          </div>
          <div class="grid grid-cols-2 gap-1 text-[9px]">
            <div
              v-for="env in envs" :key="env.id"
              class="border rounded p-1 flex flex-col gap-0.5 transition-all duration-200"
              :class="[env.color, s >= env.id ? 'opacity-100' : 'opacity-20']"
            >
              <div class="font-bold flex items-center gap-1 text-[9px]">
                <span>{{ env.icon }}</span>
                <span class="truncate">{{ env.name }}</span>
              </div>
              <div v-for="(p, pi) in env.points" :key="pi" class="text-[8px] leading-tight text-slate-600">
                • {{ p }}
              </div>
            </div>
          </div>
        </div>
      </div>

      <!-- RIGHT: The 4-Phase Heartbeat (Steps 7-13) + Synthesis (Steps 20-22) -->
      <div class="flex flex-col gap-1.5 justify-between min-h-0">
        <!-- 4 Phase Cards (Steps 7-13) -->
        <div class="border border-slate-200 rounded-lg p-2 bg-slate-50 flex flex-col gap-1">
          <div class="text-[10px] font-black text-slate-700 uppercase tracking-wider flex justify-between">
            <span>⚙️ The 4-Phase Heartbeat Cycle</span>
            <span class="text-[9px] text-amber-600 font-mono font-bold">Steps 7-13</span>
          </div>

          <div class="flex flex-col gap-1">
            <div
              v-for="p in phases" :key="p.id"
              class="border rounded-md px-2 py-1 flex items-center justify-between text-[10px] transition-all duration-200"
              :class="[
                p.color,
                s >= p.id ? 'opacity-100 translate-x-0 shadow-xs' : 'opacity-20 translate-x-2'
              ]"
            >
              <div class="flex items-center gap-1.5 truncate">
                <span class="text-sm shrink-0">{{ p.icon }}</span>
                <div>
                  <div class="font-bold leading-tight">{{ p.num }}. {{ p.label }}</div>
                  <div class="text-[8px] opacity-80 leading-tight">{{ p.detail }}</div>
                </div>
              </div>
              <span class="text-[8px] font-mono px-1 rounded bg-black/10 shrink-0 font-bold">Phase {{ p.num }}</span>
            </div>
          </div>
        </div>

        <!-- Order Matters Synthesis (Steps 20-22) -->
        <div
          class="border border-amber-300 rounded-lg p-1.5 bg-amber-50 text-amber-950 transition-all duration-300 shrink-0"
          :class="s >= 20 ? 'opacity-100 translate-y-0' : 'opacity-0 translate-y-2'"
        >
          <div class="text-[10px] font-black text-amber-900 mb-0.5 flex justify-between">
            <span>💡 Why the Order is Law:</span>
            <span class="text-[8px] bg-amber-200 text-amber-900 px-1 rounded font-bold">Steps 20-22</span>
          </div>
          <div class="text-[9px] leading-tight space-y-0.5">
            <div><span class="font-bold text-violet-900">1. Microtasks:</span> Drained to 0 immediately (Promise reactions must not wait).</div>
            <div><span class="font-bold text-rose-900">2. UI Render:</span> Screen only updates when stack is empty (prevents tearing).</div>
            <div><span class="font-bold text-amber-900">3. Macrotasks:</span> Exactly ONE per loop turn (prevents task starvation).</div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
