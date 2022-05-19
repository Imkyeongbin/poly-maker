<template>
  <div id="app">
    <PolyMaker />
  </div>
</template>

<script>
import PolyMaker from './pages/PolyMaker.vue'

export default {
  name: 'App',
  components: {
    PolyMaker
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

  min-width: 100vw;
  height: 100%;
  background-color: antiquewhite;
}
</style>
