<script setup>
import { ref, onMounted } from 'vue'
import SongItem from './components/SongItem.vue';
import Player from './components/Player.vue';
import Searchbar from './components/Searchbar.vue';

// const songs = [
//   { id: 1, title: 'Ecruteak City Theme', artist: 'Johto', src: '/songs/song1.mp3' },
//   { id: 2, title: 'RSE Surf Theme', artist: 'Hoenn', src: '/songs/song2.mp3' },
// ]

const songs = ref([])
const currentSong = ref(null)

// const offset = ref('') // The offset parameter tells the API how many items to skip before it starts returning results

const isPlaying = ref(false)

function togglePlay(song) {
  if (currentSong.value?.id === song.id) {
    isPlaying.value = !isPlaying.value
  } else {
    currentSong.value = song
    isPlaying.value = true
  }
}

function playNext() {
  const currentIndex = songs.value.findIndex(song => song.id === currentSong.value.id)

  const nextSong = songs.value[currentIndex + 1]

  if (nextSong) {
    togglePlay(nextSong)
  } else {
    fetchMoreSongs(currentSearchTerm.value)
  }
}

function playPrevious() {
  const currentIndex = songs.value.findIndex(song => song.id === currentSong.value.id)

  const previousSong = songs.value[currentIndex - 1]

  if (previousSong) {
    togglePlay(previousSong)
  }
}


const currentSearchTerm = ref('top hits')
const songListRef = ref(null)

async function fetchSongs(query) {
  currentSearchTerm.value = query
  
  try {
    const response = await fetch(`https://itunes.apple.com/search?term=${query}&media=music&limit=20`)
    const data = await response.json()

    songs.value = data.results.map(track => ({
      id: track.trackId,
      title: track.trackName,
      artist: track.artistName,
      src: track.previewUrl,
      cover: track.artworkUrl100.replace('100x100', '600x600'),
      duration: track.trackTimeMillis / 1000
    }))

    songListRef.value.scrollTop = 0 // scroll back to top
    // console.log(data)
    // console.log(songs.value)
    // offset.value = songs.value.length
  } catch (error) {
    console.warn('Error fetching more songs:', error)
  }
}


onMounted(()=>{
  fetchSongs('top hits')
})


const isLoadingMore = ref(false)
async function fetchMoreSongs(query) {
  if (isLoadingMore.value) return
  if (songs.value.length >= 100) return   // API can't return more than limit=200 (max limit)

  isLoadingMore.value = true



  const nextLimit = songs.value.length + 10

  try {
    if (query !== currentSearchTerm.value) return

    const response = await fetch(`https://itunes.apple.com/search?term=${query}&media=music&limit=${nextLimit}`)
    const data = await response.json()

    const allSongs = data.results.map(track => ({
      id: track.trackId,
      title: track.trackName,
      artist: track.artistName,
      src: track.previewUrl,
      cover: track.artworkUrl100.replace('100x100', '600x600'),
      duration: track.trackTimeMillis / 1000
    }))

    const newSongs = allSongs.filter(song => {
      return !songs.value.some(existing => existing.id === song.id)
    })

    if (newSongs.length === 0) return  // if no new songs to fetch

    songs.value = [...songs.value, ...newSongs]
    // console.log(newSongs)


  } catch (error) {
      console.warn('Error fetching more songs:', error)
  } finally {
      isLoadingMore.value = false
  }

}


function handleScroll(event) {
  const el = event.target
  const nearBottom = el.scrollHeight - el.scrollTop - el.clientHeight < 100
  if (nearBottom) {
    fetchMoreSongs(currentSearchTerm.value)
  }
}

</script>

<template>
  <div class="h-screen bg-background flex flex-col gap-6 p-4 text-text font-poppins">
    <!-- <div class="p-5"> -->
      <Searchbar @search="fetchSongs" />
    <!-- </div> -->
    <div class="bg-surface rounded-2xl overflow-hidden flex">

      <!-- BUG to fix later: Infinite scroll fails to trigger when initial content height is smaller than the container, because scroll bar doesn't appear (TO TRIGGER IT: try fetching 2 songs only)-->
      <div @scroll="handleScroll" ref="songListRef" class="p-3 flex flex-col overflow-y-auto flex-1 min-h-0 scroll-smooth
            [&::-webkit-scrollbar]:w-1.5
            [&::-webkit-scrollbar-track]:bg-transparent
            [&::-webkit-scrollbar-thumb]:bg-divider
            [&::-webkit-scrollbar-thumb]:rounded-full">

          <SongItem v-for="song in songs" :key="song.id" :song="song"
            :isPlaying="isPlaying && currentSong?.id === song.id" @play="togglePlay"></SongItem>

      </div>
    </div>

    <div class="bg-player rounded-2xl mt-auto">
      <Player :song="currentSong" :is-playing="isPlaying" @toggle-play="togglePlay(currentSong)" @next="playNext" @prev="playPrevious" @ended="isPlaying = false" />
    </div>
  </div>

</template>
