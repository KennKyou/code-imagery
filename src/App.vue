<script setup>
import Header from './components/Header.vue'
import CodeImagery from './components/CodeImagery.vue'
</script>

<template>
  <div class="app">
    <Header />
    <CodeImagery />
  </div>
</template>

<style scoped>
.app {
  display: flex;
  flex-direction: column;
  min-height: 100vh;
}

main {
  flex-grow: 1;
  padding: 1rem;
}
</style>