<template>
  <div class="week-segment w-full h-full flex items-center justify-center" :style="`--colorl: ${colorl}; --colorr: ${colorr || color}; --colorw: ${colorw}`">
    <div v-if="type === 'single'" class="custom-colorr w-3/5 rounded-full" />
    <div v-else-if="type === 'end'" class="custom-colorw w-4/5 rounded-r-full mr-auto" />
    <div v-else-if="type === 'start'" class="custom-colorw w-4/5 rounded-l-full ml-auto" />
    <div v-else-if="type === 'normal'" class="custom-colorw w-full" />
    <div v-else-if="type === 'none'" />
    <div v-else-if="type === 'both'" class="custom-colorl w-2/5 rounded-r-full mr-auto" />
    <div v-if="type === 'both'" class="custom-colorr w-2/5 rounded-l-full ml-auto" />
  </div>
</template>

<script>
import Vue from 'vue'

export default Vue.extend({
  props: {
    type: {
      type: String,
      required: true
    },
    colorl: {
      type: String,
      required: true
    },
    colorr: {
      type: String,
      default: ''
    },
    weights: {
      type: Boolean,
      default: false
    }
  },
  computed: {
    colorw () {
      return this.weights ? this.colorl : this.colorr
    }
  }
})

</script>

<style lang="postcss">
.week-segment > * {
  @apply h-3/5
}

.custom-colorl { background-color: var(--colorl); }
.custom-colorr { background-color: var(--colorr); }
.custom-colorw { background-color: var(--colorw); }
</style>
