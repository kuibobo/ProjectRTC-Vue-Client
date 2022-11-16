<template>
  <div id="app">
    <sky-rtc />
  </div>
</template>

<script>
import WebRtc from './components/WebRtc.vue'
import SkyRtc from './components/SkyRtc.vue'

export default {
  name: 'App',
  components: {
    WebRtc, SkyRtc
  }
}
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
