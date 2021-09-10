<template>
  <YouTube
    src="https://www.youtube.com/watch?v=jNQXAC9IVRw"
    @ready="onReady"
    ref="youtube"
  />
</template>

<script lang="ts">
import { defineComponent } from 'vue'
import YouTube, { Methods } from './YouTube.vue'

export default defineComponent({
  name: 'App',
  components: {
    YouTube,
  },
  methods: {
    onReady() {
      const youtube = this.$refs.youtube as Methods
      youtube?.playVideo()
    },
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
