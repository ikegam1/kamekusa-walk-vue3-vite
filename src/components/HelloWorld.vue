<script setup>
import { ref } from 'vue'
import KameCanvas from './KameCanvas.vue'

defineProps({
  msg: String
})

const count = ref(0)
</script>

<script>
export default {
  components: {
    KameCanvas
  },
  data() {
    return {
      radius: 50,
      x: 0,
      y: 0
    }
  }
}

</script>

<template>
  <h1>{{ msg }}</h1>
  <KameCanvas :radius="radius" />
  <p><input type="range" min="0" max="100" v-model.number="radius"></p>
</template>

<style scoped>
a {
  color: #42b983;
}
</style>
