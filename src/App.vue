<template>
  <div>
    <nav>
      <RouterLink to="/">Home</RouterLink>
    </nav>
    <HomeView />

    <RouterView />
  </div>
</template>

<script setup lang="ts">
import { RouterLink, RouterView } from 'vue-router'
</script>
