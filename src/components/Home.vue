<template>
  <div class="row mt-5 p-5">
    <div class="col mx-auto">
      <snake-game />
    </div>
  </div>
</template>

<script>
import SnakeGame from './SnakeGame.vue'

export default {
  name: 'HomePage',
  components: {
    SnakeGame
  },
  data() {
    return {}
  },
  methods: {}
}
</script>

<style>
#app {
  min-height: 100vh;
}
</style>
