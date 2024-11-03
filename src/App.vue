<script setup>
import Grid from './components/Grid.vue'
import Header from './components/Header.vue'
</script>

<template>
  <div id="main-container">
    <Header />
    <Grid />
  </div>
</template>

<style>
div#container {
  display: flex;
  flex-direction: column;
  justify-self: start;
}
</style>
