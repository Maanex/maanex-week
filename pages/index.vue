<template>
  <div class="week-root bg-gray-800">
    <div class="week-inner my-48">
      <input ref="area" class="week-clipboard">
      <h1 class="mb-5 text-xl font-black text-yellow-300 font-mono flex items-center">
        <img src="/icon.png" alt="" class="h-4 w-4 mr-2">
        mweek {{ getWeekId() }}
      </h1>
      <div class="w-full grid custom-grid-cols">
        <div class="px-2 py-1" />
        <div
          v-for="day of weekdays"
          :key="day"
          class="text-yellow-300 font-black px-2 py-1 text-center rounded-t"
          alternating
          v-text="day"
        />
      </div>
      <div
        v-for="task of tasks"
        :key="task.name"
        class="relative w-full grid custom-grid-cols"
        @click.middle.prevent="completeTask(task)"
      >
        <div
          class="pr-3 py-1 text-gray-300 task-name"
          @click.ctrl.prevent="deleteTask(task)"
          v-text="task.name"
        />
        <div
          v-for="(d, index) of weekdays"
          :key="d"
          class="text-yellow-300 font-black text-center"
          alternating
        >
          <segment
            :type="getSegmentType(task, index)"
            :colorl="urgencyToColor(getTaskUrgency(task, true))"
            :colorr="urgencyToColor(getTaskUrgency(task, false))"
            :weights="index < task.start"
          />
        </div>
        <div class="absolute w-0.5 h-full bg-red-400" :style="`left: ${timePosition*100}%`" />
      </div>
      <div class="relative w-full grid rounded-b custom-grid-cols">
        <div
          v-for="i of 8"
          :key="i"
          class="px-2 py-1 rounded-b h-min"
          alternating
        />
        <div class="absolute w-0.5 h-full bg-red-400" :style="`left: ${timePosition*100}%`">
          <div class="rounded-full bg-red-400 w-3 h-3 ml-px mt-0.5" style="transform: translateX(-50%)" />
        </div>
        <div
          class="pr-3 py-1 text-gray-500 task-name absolute cursor-pointer w-full"
          style="transform: translateY(30%)"
          @click="addTask()"
        >
          + Add
        </div>
      </div>
      <div class="week-spacer" />
      <div class="week-footer text-gray-700 w-100 text-center mt-20">
        <a href="https://maanex.me/" target="_blank" rel="norefferer" class="w-min transition-colors hover:text-gray-500">Made by Maanex</a>
        using
        <a href="https://nuxtjs.org/" target="_blank" rel="norefferer" class="w-min transition-colors hover:text-gray-500">Nuxt</a>
        &amp;
        <a href="https://tailwindcss.com/" target="_blank" rel="norefferer" class="w-min transition-colors hover:text-gray-500">Tailwind</a>
      </div>
    </div>
  </div>
</template>

<script>
import Vue from 'vue'

const dateZero = new Date('3 jan 2021')

export default Vue.extend({
  data () {
    return {
      timePositionUpdate: 0,
      weekdays: ['mon', 'tue', 'wed', 'thu', 'fri', 'sat', 'sun'],
      tasks: []
    }
  },
  head: {
    title: 'mweek',
    link: [{ rel: 'icon', type: 'image/x-icon', href: '/favicon.ico' }]
  },
  computed: {
    timePosition () {
      const date = new Date()
      const day = (date.getDay() + 6) % 7
      const time = (date.getHours() / 24) + (date.getMinutes() / 60 / 24)
      const pos = day + time
      return (pos + 2) / 9 + (this.timePositionUpdate * 0)
    }
  },
  mounted () {
    this.tasks = JSON.parse(localStorage.getItem('tasks') || '[{"name": "Task One","start": 0,"end": 5,"completed": 0},{"name": "Task Two","start": 3,"end": 1,"completed": 0},{"name": "Task three","start": 4,"end": -1,"completed": 0}]')

    setInterval(vue => vue.timePositionUpdate++, 10000, this)
  },
  methods: {
    copyToClipboard (text) {
      const area = this.$refs.area
      area.value = text
      area.focus()
      area.select()
      document.execCommand('copy')
    },
    getSegmentType (task, index) {
      if (index === task.start) {
        if (task.end === -1) {
          return 'single'
        }
        if (task.end === index) {
          return 'both'
        }
        return 'start'
      }
      if (index === task.end) {
        return 'end'
      }
      if (((task.start > task.end && (index > task.start || index < task.end)) ||
           (task.end > task.start && (index > task.start && index < task.end)) ||
           (task.end === task.start)) &&
           (task.end !== -1)) {
        return 'normal'
      }
      return 'none'
    },
    urgencyToColor (daysLeft) {
      switch (daysLeft) {
        case -2: return '#4b5563' // bg-gray-500 // not startable
        case -1: return '#34d399' // bg-green-400 // completed
        case 0: return '#f87171' // bg-red-400
        case 2: return '#fde68a' // bg-yellow-200
        case 1: return '#fcd34d' // bg-yellow-300
        default: return '#9ca3af' // bg-gray-400
      }
    },
    completeTask (task) {
      const weekId = this.getWeekId()
      if (task.completed === weekId) {
        task.completed = 0
      } else if (task.start >= task.end && task.end !== -1 && task.completed < weekId - 1) {
        task.completed = weekId - 1
      } else {
        task.completed = weekId
      }
      this.saveTasks()
    },
    today () {
      const d = new Date()
      const m = ['jan', 'feb', 'mar', 'apr', 'may', 'jun', 'jul', 'aug', 'sep', 'oct', 'nov', 'dec']
      return new Date(`${d.getDate()} ${m[d.getMonth()]} ${d.getFullYear()}`)
    },
    getWeekId () {
      const daysSince = Math.floor((this.today() - dateZero) / 86400000)
      return ~~(daysSince / 7)
    },
    getTaskUrgency (task, old) {
      const weekid = this.getWeekId()
      if (task.completed >= weekid) {
        return -1
      }
      const day = (new Date().getDay() + 6) % 7

      const single = task.end === -1
      if (single) {
        if (task.completed >= weekid) { return -1 }
        if (task.start <= day) { return 0 }
        return -2
      }

      const flipped = task.start >= task.end
      if (flipped) {
        if (old) {
          if (task.completed >= weekid - 1) { return -1 }
          if (day > task.end) { return 0 }
          return (task.end + 7 - day) % 7
        } else {
          if (day < task.start) { return -2 }
          return (task.end + 6 - day) % 7 + 1
        }
      }

      if (task.start > day) { return -2 }
      if (task.end < day) { return 0 }

      return (task.end + 7 - day) % 7
    },
    saveTasks () {
      localStorage.setItem('tasks', JSON.stringify(this.tasks))
    },
    deleteTask (task) {
      if (!confirm(`delete task ${task.name}?`)) {
        return
      }
      const index = this.tasks.findIndex(e => e.name === task.name)
      this.tasks.splice(index, 1)
      Vue.nextTick(() => this.saveTasks())
    },
    addTask () {
      if (!confirm('add a new task? you can remove tasks by clicking on their name while holding down the ctrl key. middleclick a task to toggle it between completed and todo. continue?')) {
        return
      }
      const name = window.prompt('what\'s the task called?')
      let start = NaN
      while (isNaN(start)) {
        const input = window.prompt('when can you start working on it? you can use mon, tue, wed, thu, fri, sat, sun. type cancel to cancel.')
        start = ['cancel', 'mon', 'tue', 'wed', 'thu', 'fri', 'sat', 'sun'].indexOf(input.toLowerCase()) - 1
        if (start === -1) { return }
      }
      let end = NaN
      while (isNaN(end)) {
        const input = window.prompt('when is the last day to work on it? you can use mon, tue, wed, thu, fri, sat, sun. type cancel to cancel.')
        end = ['cancel', 'mon', 'tue', 'wed', 'thu', 'fri', 'sat', 'sun'].indexOf(input.toLowerCase()) - 1
        if (end === -1) { return }
      }
      if (start === end) {
        const input = window.prompt('start and end day are the same. is this a task that takes the entire week or that one single day? type week or day.')
        if (input.toLowerCase() === 'day') { end = -1 }
      }
      this.tasks.push({
        name,
        start,
        end,
        completed: 0
      })
      Vue.nextTick(() => this.saveTasks())
    }
  }
})

</script>

<style lang="postcss">
.week-root {
  width: 100vw;
  height: 100vh;
  position: absolute;
  top: 0;
  left: 0;
  margin: 0;
  padding: 0;
  overflow-x: hidden;
  display: flex;
  justify-content: center;
}

.week-inner {
  width: 90vw;
  max-width: 600px;
  display: flex;
  flex-direction: column;
}

.week-clipboard {
  opacity: 0;
  height: 0;
  pointer-events: none;
}

.week-spacer {
  flex-grow: 1;
  flex-shrink: 1;
}

.week-footer * {
  color: inherit;
}

[alternating]:nth-child(2n) { @apply bg-gray-700 }

.custom-grid-cols {
  grid-template-columns: 2fr repeat(7, 1fr)
}

.custom-grid-cols:hover .week-segment {
  filter: brightness(1.1);
  box-shadow: 0 2 2px -2px #00000055;
}
.custom-grid-cols:hover .task-name {
  @apply text-gray-100
}
</style>
