<script setup lang="ts">
import { RouterLink, RouterView } from 'vue-router'
</script>

<template>
  <header class="bg-slate-800 text-white">
    <nav class="max-w-3xl lg:max-w-7xl mx-auto py-6 text-xl px-8">
      <RouterLink to="/">Home</RouterLink>
    </nav>
  </header>

  <main class="max-w-3xl lg:max-w-7xl mx-auto py-6 px-8">
    <RouterView />
  </main>
</template>
