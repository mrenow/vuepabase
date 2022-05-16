<script setup>
// This starter template is using Vue 3 <script setup> SFCs
// Check out https://v3.vuejs.org/api/sfc-script-setup.html#sfc-script-setup
import HelloWorld from './components/HelloWorld.vue'
import Auth from './components/Auth.vue'
import NewBlog from './components/NewBlog.vue'
import Lights from './components/lights_out/Lights.vue'

import { ref, computed } from 'vue'

const routesMap = {
  '': HelloWorld,
  'auth': Auth,
  'new-blog': NewBlog,
  'lights': Lights
}


const currentPath = ref(window.location.pathname)

window.addEventListener('hashchange', () => {
  currentPath.value = window.location.pathname
})

console.log(currentPath.value.slice(1))

const currentView = computed(() => {
  return routesMap[currentPath.value.slice(1)]
})

</script>

<template>
  <div class=content-area>
  <a href="/">Home</a> 
  <a href="/auth">About</a> 
  <a href="/new-blog">Broken Link</a>
  <!--HelloWorld msg="Hello Vue 3 + Vite" /-->
  <component :is='currentView'/>
  </div>
</template>

<style>
/* 
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
} */
</style>
