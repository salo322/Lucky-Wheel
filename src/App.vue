<template>
  <div id="app">
    <LuckyWheel />
  </div>
</template>

<script>
import LuckyWheel from './components/LuckyWheel.vue';

export default {
  components: {
    LuckyWheel
  }
};
</script>

<style>
body{
  background-color: #ECDFCC;
}
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}
</style>
