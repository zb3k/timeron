<script setup>
import { computed, reactive, ref } from 'vue';
import Time from './components/Time.vue';

const exportMode = ref(false);
const activeTimer = ref({});
const editMode = ref(false);
let activeTimerBegintime;
let timerInterval;
const timers = reactive(loadData());
const newTimerName = ref('');
const exportContent = computed(() => {
  return timers.map(({ time, name }) => Math.round(time / 60) + '\t' + name).join('\n');
});
const totalTime = computed(() => {
  return timers.map(({ time }) => time).reduce((a, b) => a + b, 0);
});


function loadData() {
  let data = localStorage.getItem('timers');

  return data ? JSON.parse(data) : [];
}

function saveData() {
  localStorage.setItem('timers', JSON.stringify(timers));
}

function unixtime() {
  return parseInt(new Date().getTime() / 1000)
}

function timerTick() {
  const time = unixtime() - activeTimerBegintime;
  activeTimer.value.time = time;
}


function toggleTimer(timer, status = null) {
  if (activeTimer.value && (activeTimer.value.name == timer.name && !status)) {
    stopTimer();
  } else {
    activeTimer.value = timer;
    activeTimerBegintime = unixtime() - timer.time;
    timerInterval = setInterval(timerTick, 1000);
  }
}

function addTimer() {
  if (!newTimerName.value) {
    return;
  }
  let timer = {
    time: 0,
    name: newTimerName.value
  };
  newTimerName.value = '';
  timers.push(timer);
  toggleTimer(timer)
  saveData();
}

function stopTimer() {
  activeTimer.value = {};
  if (timerInterval) {
    clearInterval(timerInterval);
  }
  saveData();
}
function deleteTimer(index) {
  timers.splice(index, 1);
  saveData();
}
function deleteAll() {
  if (confirm('Delete all timers?')) {
    timers.splice(0);
    saveData();
  }
}
function togglExport() {
  editMode.value = false;
  exportMode.value = !exportMode.value;
}
function togglEdit() {
  exportMode.value = false;
  editMode.value = !editMode.value;
}

const htmlNode = document.body.parentNode;
const currentTheme = ref(htmlNode.className);
const themeDark = 'theme-dark';
const themeLight = 'theme-light';
const isDarkTheme = computed(() => currentTheme.value == themeDark);

function toggleTheme() {
  currentTheme.value = currentTheme.value == themeDark ? themeLight : themeDark;
  htmlNode.className = currentTheme.value;
  localStorage.setItem('theme', currentTheme.value);
}

</script>

<template>
  <div class="toolbar" :class="{ 'toolbar__active': editMode || exportMode || activeTimer.name }">
    <div class="toolbar__left">
      <!-- <button @click="stopTimer" :disabled="!activeTimer.name">Stop</button> -->
      <button @click="togglEdit" :class="{ active: editMode }" :disabled="timers.length == 0">EDIT</button>
      <button @click="togglExport" :class="{ active: exportMode }" :disabled="timers.length == 0">EXPORT</button>
    </div>

    <button @click="stopTimer" :class="{ active: activeTimer.name }" :disabled="timers.length == 0">
      <Time :class="{ active: activeTimer.name }" :seconds="totalTime" with-labels />
    </button>


    <div class="toolbar__right">
      <button @click="toggleTheme">{{ currentTheme === themeDark ? 'LIGHT' : 'DARK' }}</button>
      <button @click="deleteAll" :disabled="timers.length == 0">RESET</button>
    </div>
  </div>

  <textarea v-if="exportMode" class="export" readonly>{{ exportContent }}</textarea>
  <div v-else class="timers">
    <div v-for=" timer, index  in  timers " :key="index" class="timer-row"
      :class="{ active: timer.name === activeTimer.name }" @click="toggleTimer(timer)">
      <div class="timer-row__time" v-if="!editMode"><Time :seconds="timer.time" /></div>
      <div class="timer-row__name">
        <input v-if="editMode" type="text" v-model="timer.name" @click.stop @focus="toggleTimer(timer, true)"
          @blur="saveData">
        <span v-else>{{ timer.name }}</span>
      </div>
      <div class="timer-row__actions">
        <span class="icon" v-if="!editMode">{{ timer.name === activeTimer.name ? "❚❚" : "▶" }}</span>

        <button v-if="editMode" @click.stop="deleteTimer(index)">❌</button>
      </div>
    </div>
    <form @submit.prevent="addTimer">
      <input type="text" v-model="newTimerName" placeholder="Write timer name...">
    </form>
  </div>
</template>

