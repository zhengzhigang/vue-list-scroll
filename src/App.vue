<template>
  <div id="app">
    <!-- <img alt="Vue logo" src="./assets/logo.png"> -->
    <ListScroll />
  </div>
</template>

<script>
import ListScroll from './components/ListScroll.vue'

export default {
  name: 'App',
  components: {
    ListScroll
  },
  methods: {
    renderFun() {

    }
  }
}
</script>

<style>
body {
  margin: 0;
}
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}
</style>
