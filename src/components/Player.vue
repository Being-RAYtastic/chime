<script setup>
import { ref, watch, computed } from 'vue'
import { Play, Pause, SkipBack, SkipForward, Heart } from 'lucide-vue-next'

const props = defineProps(['song', 'isPlaying', 'isLiked'])
const emit = defineEmits(['togglePlay', 'next', 'prev', 'ended', 'like'])

const audioPlayer = ref(null)

const currentTime = ref(0)
const duration = ref(0)


watch(() => props.isPlaying, (playing) => {
    if (!audioPlayer.value) return  // Guard if there is no song

    if (playing) audioPlayer.value.play()
    else audioPlayer.value.pause()
}, { flush: 'post' })



function formatTime(seconds) {
    if (!seconds || isNaN(seconds)) return '00:00'
    const minutes = Math.floor(seconds / 60)
    const paddedMinutes = minutes.toString().padStart(2, '0')

    const remainingSeconds = Math.floor(seconds % 60)
    const paddedSeconds = remainingSeconds.toString().padStart(2, '0')

    return `${paddedMinutes}:${paddedSeconds}`
}


function seek(event) {
    const newTime = Number(event.target.value)
    audioPlayer.value.currentTime = newTime
    currentTime.value = newTime
}

const progressPercent = computed(() => {
  if (!duration.value) return 0
  return (currentTime.value / duration.value) * 100
})

</script>


<template>
    <!-- <div v-if="song" class="flex justify-between w-full items-center"> -->
    <div class="grid grid-cols[1fr_2fr] sm:grid sm:grid-cols-[1fr_2fr_1fr] items-center w-full gap-4 p-3">
        <div v-if="song" class="flex items-center gap-3 min-w-0">
            <section v-if="song" class="w-full flex justify-between">
                <div class="flex items-center gap-3 min-w-0">
                    <img :src="song.cover" class="w-12 h-12 rounded-md object-cover shrink-0 block" />
                    <p class="text-sm text-white truncate">{{ song.title }}</p>
                </div>
                <div class="sm:hidden flex justify-end mr-5 items-center">
                    <button @click="emit('like', song)" class="active:scale-90 transition-all duration-200">
                        <Heart class="w-6 h-6 cursor-pointer transition-all duration-200" :class="isLiked? 'text-accent-secondary fill-accent-secondary': 'text-text'"></Heart>
                    </button>
                </div>
            </section>
        </div>
        <div v-else></div>

        <div class="flex flex-col justify-center items-stretch gap-2">
            <div class="flex items-center gap-3">
                <span class="text-xs text-muted">{{ formatTime(Math.round(currentTime)) }}</span>
                <input
                    type="range"
                    min="0"
                    :max="Math.round(duration)"
                    :value="currentTime"
                    @input="seek"
                    :disabled="!song"
                    :style="{ background: `linear-gradient(to right, var(--color-accent) ${progressPercent}%, var(--color-divider) ${progressPercent}%)`, transition: `'background 0.05s ease-in'` }"
                    class="w-full h-1 rounded-full appearance-none cursor-pointer disabled:opacity-50 disabled:cursor-not-allowed
                        [&::-webkit-slider-thumb]:appearance-none
                        [&::-webkit-slider-thumb]:w-0
                        [&::-webkit-slider-thumb]:h-0
                        [&::-webkit-slider-thumb]:rounded-full
                        [&::-webkit-slider-thumb]:bg-accent"
                    />
                <span class="text-xs text-muted">{{ formatTime(Math.round(duration)) }}</span>
            </div>
            <audio 
                ref="audioPlayer" 
                :src="song?.src" 
                autoplay
                @timeupdate="currentTime = audioPlayer.currentTime"
                @loadedmetadata="duration = audioPlayer.duration"
                @ended="emit('ended')"
                >
            </audio>
            <div class="m-auto flex items-center gap-5 text-white">
                <button @click="emit('prev')" :disabled="!song" class="disabled:opacity-40 disabled:cursor-not-allowed">
                    <SkipBack class="w-4 h-4 cursor-pointer" />
                </button>
                <button @click="emit('togglePlay')" :disabled="!song" class="disabled:opacity-40 disabled:cursor-not-allowed">
                    <Pause v-if="isPlaying" class="w-5 h-5 cursor-pointer text-accent" />
                    <Play v-else class="w-5 h-5 cursor-pointer" />
                </button>
                <button @click="emit('next')" :disabled="!song" class="disabled:opacity-40 disabled:cursor-not-allowed">
                    <SkipForward class="w-4 h-4 cursor-pointer" />
                </button>
            </div>
        </div>
        <div class="hidden sm:flex justify-end mr-5 items-center">
            <button v-if="song" @click="emit('like', song)" class="active:scale-90 transition-all duration-200">
                <Heart class="w-6 h-6 cursor-pointer transition-all duration-200" :class="isLiked? 'text-accent-secondary fill-accent-secondary': 'text-text'"></Heart>
            </button>
        </div>
    </div>
    
</template>