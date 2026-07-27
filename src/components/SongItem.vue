<script setup>

import { Play, Pause, SkipBack, SkipForward } from 'lucide-vue-next'


defineProps(['song', 'isPlaying'])
const emit = defineEmits(['play'])

function togglePlay() {


  if (!props.isPlaying) {
    audioPlayer.value.play()
  } else {
    audioPlayer.value.pause()
  }

  // Notify App.vue to flip the boolean state
  emit('togglePlay')

}



function formatTime(seconds) {
  if (!seconds || isNaN(seconds)) return '00:00'
  const minutes = Math.floor(seconds / 60)
  const paddedMinutes = minutes.toString().padStart(2, '0')

  const remainingSeconds = Math.floor(seconds % 60)
  const paddedSeconds = remainingSeconds.toString().padStart(2, '0')

  return `${paddedMinutes}:${paddedSeconds}`
}

</script>

<template>

  <!-- <li class="flex items-center justify-between bg-gray-100 text-black rounded-full px-4 py-2 mb-3"> -->
  <div @click="emit('play', song)" class="flex items-center gap-3 px-2 py-2.5 border-b  border-divider cursor-pointer"
    :class="isPlaying ? 'bg-accent/10 rounded-lg border-transparent' : ''">
    <!-- <div class="w-10 h-8 rounded-full bg-gray-400"> -->
    <img :src="song.cover" class="w-10 h-10 rounded-md object-cover shrink-0 block">
    <!-- </div> -->
    <div class="flex-1 min-w-0">
      <p class="text-sm truncate" :class="isPlaying ? 'text-accent font-medium' : 'text-text'">
        {{ song.title }}
      </p>
      <p class="text-xs truncate" :class="isPlaying ? 'text-accent/70' : 'text-muted'">
        {{ song.artist }}
      </p>
    </div>
    <span class="text-xs" :class="isPlaying ? 'text-accent' : 'text-muted'">
      {{ formatTime(song.duration) }}
    </span>

    <div class="mr-3">
      <Pause v-if="isPlaying" class="w-4 h-4 cursor-pointer" :class="isPlaying ? 'text-accent' : 'text-muted'"
        fill="currentColor" :stroke-width="0.5" />
      <Play v-else class="w-4 h-4 cursor-pointer text-muted" />
    </div>

  </div>
  <!-- </li> -->

</template>