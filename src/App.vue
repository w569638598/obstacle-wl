<template>
  <div id="app" ref="app">
    <router-view></router-view>
    <!-- <HelloWorld msg="Welcome to Your Vue.js App"/> -->
  </div>
</template>

<script>
// import HelloWorld from '../packages/x6'

export default {
  name: 'App',
  components: {
    // HelloWorld
  },
  mounted(){
    this.$refs.app.style.cssText = "height: " + window.innerHeight + "px"
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
}
#app > div{
  height: 100%;
}
body{
  margin: 0;
}
</style>
