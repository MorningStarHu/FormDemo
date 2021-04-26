<!--
 * @Descripttion: 
 * @version: 
 * @Author: morningtarHU
 * @Date: 2021-04-06 15:16:41
 * @LastEditors: morningtarHU
 * @LastEditTime: 2021-04-08 14:39:02
-->
<template>
  <div id="app">
    <img src="./assets/logo.png">
    <Example></Example>
  </div>
</template>

<script>
import Example from './components/form/index.vue'

export default {
  name: 'App',
  components: {
    Example
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
