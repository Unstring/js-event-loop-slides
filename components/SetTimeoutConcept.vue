<script setup lang="ts">
import { computed } from 'vue'
const props = defineProps<{ step?: number }>()
const s = computed(() => Math.min(Math.max(props.step ?? 0, 0), 22))

const whatItDoes = [
  { id: 1, text: 'NOT a sleep() — JS thread is never blocked', icon: '❌', bad: true },
  { id: 2, text: 'Registers a timer with the host environment (browser C++ / libuv)', icon: '📞', bad: false },
  { id: 3, text: 'Host environment tracks timer on a separate OS thread', icon: '🧵', bad: false },
  { id: 4, text: 'When delay expires → callback pushed to Macrotask Queue', icon: '📬', bad: false },
  { id: 5, text: 'Callback only runs when Stack is EMPTY and microtasks are drained', icon: '⏳', bad: false },
  { id: 6, text: 'Specified delay = MINIMUM time, never guaranteed exact time', icon: '⚠️', bad: true },
]

const heapItems = [
  { id: 8, timer: 'Timer A', delay: '10ms', expires: 'T+10', pos: 'Root (soonest)', color: 'bg-emerald-100 border-emerald-400' },
  { id: 9, timer: 'Timer B', delay: '50ms', expires: 'T+50', pos: 'Level 2', color: 'bg-sky-100 border-sky-300' },
  { id: 10, timer: 'Timer C', delay: '100ms', expires: 'T+100', pos: 'Level 3', color: 'bg-violet-100 border-violet-300' },
]

const clampingRules = [
  { id: 19, text: 'setTimeout(fn, 0) actually fires at 0ms (first call)', tag: 'OK', color: 'bg-emerald-50 border-emerald-300 text-emerald-900' },
  { id: 20, text: 'Nested setTimeout at depth ≥ 5 → clamped to 4ms minimum (HTML5 spec)', tag: '4ms rule', color: 'bg-amber-50 border-amber-400 text-amber-900' },
  { id: 21, text: 'Background/hidden tab → throttled to 1000ms (1 second!) to save battery', tag: 'Tab throttle', color: 'bg-orange-50 border-orange-400 text-orange-900' },
  { id: 22, text: 'requestAnimationFrame paused entirely in hidden tabs', tag: 'rAF stops', color: 'bg-rose-50 border-rose-400 text-rose-900' },
]
</script>

<template>
  <div class="h-full flex flex-col gap-2 select-none">
    <div class="flex items-center gap-3 pb-1 border-b-2 border-slate-200">
      <span class="text-2xl">⏱️</span>
      <div>
        <h2 class="text-xl font-black text-slate-900 leading-tight">setTimeout Internals</h2>
        <p class="text-xs text-slate-500">Chapter 4 of 5 · Concept</p>
      </div>
      <div class="ml-auto px-2 py-1 rounded-lg bg-sky-600 text-white text-xs font-bold">Step {{ s }}/22</div>
    </div>

    <div class="grid grid-cols-2 gap-3 flex-1">
      <!-- LEFT -->
      <div class="flex flex-col gap-2">
        <div class="text-xs font-black text-slate-700 uppercase tracking-wider">📌 What setTimeout Actually Does</div>
        <div class="flex flex-col gap-1">
          <div
            v-for="item in whatItDoes" :key="item.id"
            class="border-2 rounded-lg px-2.5 py-1.5 text-xs font-medium flex items-center gap-2 transition-all duration-400"
            :class="[
              item.bad ? 'bg-rose-50 border-rose-300 text-rose-900' : 'bg-slate-50 border-slate-300 text-slate-800',
              s >= item.id ? 'opacity-100 translate-x-0' : 'opacity-0 -translate-x-4'
            ]"
            style="transition: opacity 0.35s, transform 0.35s"
          >
            <span class="text-base shrink-0">{{ item.icon }}</span>
            <span>{{ item.text }}</span>
          </div>
        </div>

        <!-- Code comparison -->
        <div class="mt-1" :class="s >= 13 ? 'opacity-100' : 'opacity-0'" style="transition: opacity 0.4s">
          <div class="text-xs font-black text-rose-700 uppercase tracking-wider mb-1">💥 The 0ms Myth</div>
          <div class="bg-slate-900 rounded-xl p-3 font-mono text-[11px] leading-relaxed">
            <div class="text-slate-500 text-[10px] mb-1">// What you think:</div>
            <div :class="s >= 13 ? 'text-rose-300' : 'text-slate-600 opacity-0'" style="transition: opacity 0.3s">setTimeout(fn, 0) // "runs immediately"</div>
            <div class="text-slate-500 text-[10px] mt-2 mb-1">// What actually happens:</div>
            <div :class="s >= 14 ? 'text-amber-300' : 'text-slate-600 opacity-0'" style="transition: opacity 0.3s">// 1. Timer registered with host</div>
            <div :class="s >= 15 ? 'text-amber-300' : 'text-slate-600 opacity-0'" style="transition: opacity 0.3s delay-100ms">// 2. Stack must be empty first</div>
            <div :class="s >= 16 ? 'text-amber-300' : 'text-slate-600 opacity-0'" style="transition: opacity 0.3s delay-200ms">// 3. ALL microtasks must drain</div>
            <div :class="s >= 17 ? 'text-emerald-300 font-black' : 'text-slate-600 opacity-0'" style="transition: opacity 0.3s delay-300ms">// THEN fn() runs — could be 2000ms later!</div>
          </div>
        </div>
      </div>

      <!-- RIGHT -->
      <div class="flex flex-col gap-2">
        <!-- Min-Heap -->
        <div :class="s >= 7 ? 'opacity-100' : 'opacity-0'" style="transition: opacity 0.4s">
          <div class="text-xs font-black text-slate-700 uppercase tracking-wider mb-1">🌳 Min-Heap Timer Priority Queue</div>
          <div class="bg-slate-50 border-2 border-slate-200 rounded-xl p-3">
            <div class="text-[10px] text-slate-600 mb-2" :class="s >= 7 ? 'opacity-100' : 'opacity-0'">
              Browser/libuv stores all timers in a Min-Heap — O(log n) insertion, root = soonest timer.
            </div>
            <div class="flex flex-col gap-1.5">
              <div
                v-for="item in heapItems" :key="item.id"
                class="border-2 rounded-lg px-2.5 py-1.5 flex items-center justify-between text-xs transition-all duration-400"
                :class="[item.color, s >= item.id ? 'opacity-100 scale-100' : 'opacity-0 scale-95']"
                style="transition: opacity 0.35s, transform 0.35s"
              >
                <div>
                  <div class="font-black">{{ item.timer }}</div>
                  <div class="text-[10px] opacity-80">{{ item.pos }}</div>
                </div>
                <div class="text-right">
                  <div class="font-black font-mono">{{ item.delay }}</div>
                  <div class="text-[10px] opacity-80">{{ item.expires }}</div>
                </div>
              </div>
            </div>
            <div class="text-[10px] text-slate-600 mt-2" :class="s >= 11 ? 'opacity-100' : 'opacity-0'" style="transition: opacity 0.3s">
              OS sets a single hardware interrupt for the root timer. When it fires → callback enters Macrotask Queue.
            </div>
          </div>
        </div>

        <!-- Clamping rules -->
        <div :class="s >= 19 ? 'opacity-100' : 'opacity-0'" style="transition: opacity 0.4s">
          <div class="text-xs font-black text-slate-700 uppercase tracking-wider mb-1">📋 HTML5 Clamping & Throttle Rules</div>
          <div class="flex flex-col gap-1">
            <div
              v-for="rule in clampingRules" :key="rule.id"
              class="border-2 rounded-lg px-2.5 py-1 flex items-center justify-between text-xs transition-all duration-400"
              :class="[rule.color, s >= rule.id ? 'opacity-100 translate-x-0' : 'opacity-0 translate-x-4']"
              style="transition: opacity 0.35s, transform 0.35s"
            >
              <span class="flex-1 pr-2">{{ rule.text }}</span>
              <span class="shrink-0 font-black text-[10px] bg-white/60 px-1.5 py-0.5 rounded border">{{ rule.tag }}</span>
            </div>
          </div>
        </div>

        <!-- Key takeaway -->
        <div
          class="mt-auto bg-amber-50 border-2 border-amber-400 rounded-xl px-3 py-2 transition-all duration-500"
          :class="s >= 18 ? 'opacity-100' : 'opacity-0'"
        >
          <div class="text-xs font-black text-amber-900">💡 The Real Model</div>
          <div class="text-[11px] text-amber-900 mt-1">setTimeout(fn, delay) = <strong>"minimum delay"</strong>. The actual time depends on: stack depth, microtask backlog, other queued macrotasks, and tab visibility.</div>
        </div>
      </div>
    </div>
  </div>
</template>
