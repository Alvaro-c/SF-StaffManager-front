<template>
  <div>
    <nav class="nav"><!-- <RouterLink to="/">Home</RouterLink> -->SF staff manager</nav>
    <Suspense>
      <HomeView />
    </Suspense>
  </div>
</template>

<script setup lang="ts">
import HomeView from './views/HomeView.vue'
// import { RouterLink } from 'vue-router'
</script>

<style>
.nav {
  width: 100%;
  height: 5%;
  background-color: rgba(255, 251, 0, 0.9);
  color: black;
  text-align: center;
  font-size: 50px;
  padding: 5px 0 10px 0;
}
</style>
