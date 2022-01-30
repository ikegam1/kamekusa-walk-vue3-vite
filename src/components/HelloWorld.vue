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
  }
}

</script>

<template>
  <h1>{{ msg }}</h1>
  <KameCanvas />
</template>

<style scoped>
a {
  color: #42b983;
}
</style>
