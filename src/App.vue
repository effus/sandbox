<template>
  <div id="app">
    <sandbox-container></sandbox-container>
  </div>
</template>

<script>
//import HelloWorld from './components/HelloWorld.vue'
import sandboxContainer from './view/sandboxContainer.vue';

export default {
  name: 'App',
  components: {sandboxContainer}
}
</script>

<style lang="scss">
body, html {
  width: 100%;
  height: 100%;
  margin: 0;
  padding: 0;
}
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  background: url('/img/bg-01.png');
  background-position-y: 5%;
  width: 100%;
  height: 100%;
}
</style>
