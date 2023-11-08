<script setup>
import { reactive, ref, watch, watchEffect } from 'vue'

const props = defineProps({
  seconds: Number,
  withLabels: Boolean,
})

const time = reactive({ hours: 0, minutes: 0, seconds: 0, HH: '00', MM: '00', SS: '00' })


watchEffect(() => {
  let sec_num = parseInt(props.seconds, 10);
  time.hours = Math.floor(sec_num / 3600);
  time.minutes = Math.floor((sec_num - (time.hours * 3600)) / 60);
  time.seconds = sec_num - (time.hours * 3600) - (time.minutes * 60);

  time.HH = (time.hours < 10 ? "0" : "") + time.hours;
  time.MM = (time.minutes < 10 ? "0" : "") + time.minutes;
  time.SS = (time.seconds < 10 ? "0" : "") + time.seconds;
});



</script>

<template>
  <span class="time">
    <span class="time__hours" :class="{ muted: !time.hours }">{{ time.hours }}<span v-if="withLabels">h</span></span>
    <span class="time__minutes" :class="{ muted: !time.minutes && !time.hours }">{{ time.minutes }}<span
        v-if="withLabels">m</span></span>
    <span class="time__seconds">{{ time.seconds }}<span v-if="withLabels">s</span></span>
  </span>
</template>

<style>
</style>
