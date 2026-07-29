<script setup>
import { useRoute } from 'vue-router'

const route = useRoute()

</script>

<template>
  <nav class="flex gap-2 justify-around items-center py-1">
    <router-link
      to="/"
      class="px-4 py-2 rounded-2xl text-sm font-medium transition-colors"
      :class="route.path === '/' 
        ? 'text-accent' 
        : 'text-muted hover:text-text'"
    >
      Home
    </router-link>

    <router-link
      to="/liked"
      class="px-4 py-2 rounded-2xl text-sm font-medium transition-colors"
      :class="route.path === '/liked' 
        ? 'text-accent' 
        : 'text-muted hover:text-text'"
    >
      Liked
    </router-link>
  </nav>
</template>