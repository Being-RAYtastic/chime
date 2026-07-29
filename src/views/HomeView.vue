<script setup>
import SongItem from '../components/SongItem.vue'

defineProps(['songs', 'currentSong', 'isPlaying'])
const emit = defineEmits(['play'])

</script>



<template>
  <SongItem v-for="song in songs" :key="song.id" :song="song" :isPlaying="isPlaying && currentSong?.id === song.id"
    @play="emit('play', song)" />
</template>