<template>
  <YouTube src="https://www.youtube.com/watch?v=jNQXAC9IVRw" />
</template>

<script lang="ts">
import { defineComponent } from 'vue'
import YouTube from './YouTube.vue'

export default defineComponent({
  name: 'App',
  components: {
    YouTube,
  },
})
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
