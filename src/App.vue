<template>
  <div id="app">
    <HelloWorld/>
  </div>
</template>

<script>
import HelloWorld from './components/HelloWorld.vue'

export default {
  name: 'App',
  components: {
    HelloWorld
  }
}
</script>

<style lang="scss">
@import "destyle.css/destyle";
@import "@/styles/variables";

body {
  background-color: $background;
  font-family: 'Montserrat', sans-serif;
}
</style>