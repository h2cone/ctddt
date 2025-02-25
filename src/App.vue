<script setup lang="ts">
import { computed, onMounted, reactive, ref, nextTick } from 'vue';

const countdown = ref();
const time = ref(1);
const isReady = ref(false);

const initSlot = () => {
  return {
    seconds: 0,
    minutes: 0,
    hours: 0,
    days: 0,
  }
}
const slot = reactive(initSlot());

const seconds = computed(() => {
  return { '--value': slot.seconds, }
});
const minutes = computed(() => ({
  '--value': slot.minutes,
}));
const hours = computed(() => ({
  '--value': slot.hours,
}));
const days = computed(() => ({
  '--value': slot.days,
}));

function onProgress(data: any) {
  slot.seconds = data.seconds;
  slot.minutes = data.minutes;
  slot.hours = data.hours;
  slot.days = data.days;
}

enum Switch {
  Play = 'Play',
  Pause = 'Pause',
}
const initSate = () => {
  return {
    running: false,
    paused: false,
    ended: true,
    switch: Switch.Play
  }
}
const state = reactive(initSate());

let last = 0;
const daysInput = ref();
const hoursInput = ref();
const minutesInput = ref();
const secondsInput = ref();

function getTimeValue() {
  const days = daysInput.value.value;
  const hours = hoursInput.value.value;
  const minutes = minutesInput.value.value;
  const seconds = secondsInput.value.value;
  return last === 0 ? (seconds * 1000 + minutes * 60 * 1000 + hours * 60 * 60 * 1000 + days * 24 * 60 * 60 * 1000) + 1000 : last;
}

async function onSwitch() {
  if (state.running) {
    countdown.value.abort();
    last = countdown.value.totalMilliseconds;

    state.running = false;
    state.paused = true;
    state.ended = false;
    state.switch = Switch.Play;
  } else {
    state.running = true;
    state.paused = false;
    state.ended = false;
    state.switch = Switch.Pause;
    await nextTick();

    time.value = getTimeValue();
    countdown.value.restart();
  }
}

const NOTIFICATION_TITLE = 'countdown-desktop';
const NOTIFICATION_BODY = 'Your countdown has ended!'

function onCancel() {
  reset();
  countdown.value.end();
}

function reset() {
  Object.assign(state, initSate());
  Object.assign(slot, initSlot());
  last = 0;
}

function onEnd() {
  if (!state.ended) {
    reset();
  }
  new window.Notification(NOTIFICATION_TITLE, { body: NOTIFICATION_BODY });
}

onMounted(() => {
  isReady.value = true;
});
</script>

<template>
  <vue-countdown v-if="isReady" ref="countdown" :auto-start="false" :time="time" @progress="onProgress" @end="onEnd">
    <div class="flex justify-center items-center flex-col h-screen">
      <div class="grid grid-flow-col gap-5 text-center auto-cols-max" v-show="state.running || state.paused">
        <div class="flex flex-col p-2 bg-neutral rounded-box text-neutral-content">
          <span class="countdown font-mono text-5xl">
            <span :style="days"></span>
          </span>
          days
        </div>
        <div class="flex flex-col p-2 bg-neutral rounded-box text-neutral-content">
          <span class="countdown font-mono text-5xl">
            <span :style="hours"></span>
          </span>
          hours
        </div>
        <div class="flex flex-col p-2 bg-neutral rounded-box text-neutral-content">
          <span class="countdown font-mono text-5xl">
            <span :style="minutes"></span>
          </span>
          min
        </div>
        <div class="flex flex-col p-2 bg-neutral rounded-box text-neutral-content">
          <span class="countdown font-mono text-5xl">
            <span :style="seconds"></span>
          </span>
          sec
        </div>
      </div>
      <div class="flex mt-5 gap-1" v-show="!(state.running || state.paused)">
        <input ref="daysInput" class="input input-bordered input-md w-full max-w-xs" type="number" min="0" max="99"
          oninput="if(value<0)value=0; if(value>99)value=99" placeholder="Days">

        <input ref="hoursInput" class="input input-bordered input-md w-full max-w-xs" type="number" min="0" max="23"
          oninput="if(value<0)value=0; if(value>23)value=23;" placeholder="Hours">

        <input ref="minutesInput" class="input input-bordered input-md w-full max-w-xs" type="number" min="0" max="59"
          oninput="if(value<0)value=0; if(value>59)value=59;" placeholder="Minutes">

        <input ref="secondsInput" class="input input-bordered input-md w-full max-w-xs" type="number" min="0" max="59"
          oninput="if(value<0)value=0; if(value>59)value=59;" placeholder="Seconds">
      </div>
      <div class="flex mt-5 gap-5">
        <button class="btn btn-wide" @click="onCancel" :disabled="state.ended">Cancel</button>
        <button class="btn btn-wide" @click="onSwitch">{{ state.switch }}</button>
      </div>
    </div>
  </vue-countdown>
</template>