<template>
  <div id="app">
    <a href="https://valanter.com"><img src="./assets/logo-valanter.png"></a>
    <SimpleStorage/>
  </div>
</template>

<script>
import SimpleStorage from './components/SimpleStorage'

export default {
  name: 'App',
  components: {
    SimpleStorage
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
