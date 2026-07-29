<script setup>
import SongItem from '../components/SongItem.vue'

defineProps(['likedSongs', 'currentSong', 'isPlaying'])
const emit = defineEmits(['play'])
</script>

<template>
    <SongItem v-for="song in likedSongs" :key="song.id" :song="song" :isPlaying="isPlaying && currentSong?.id === song.id"
        @play="emit('play', song)" />
</template>