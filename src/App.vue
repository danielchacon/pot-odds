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

body {
  background-color: #f2f2f2;
  font-family: 'Montserrat', sans-serif;
}
</style>