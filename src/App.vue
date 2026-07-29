<script setup>
import { ref, onMounted, computed } from 'vue'
import { useRoute, useRouter } from 'vue-router'

import SongItem from './components/SongItem.vue';
import Player from './components/Player.vue';
import Searchbar from './components/Searchbar.vue';
import Navbar from './components/Navbar.vue';

// const songs = [
//   { id: 1, title: 'Ecruteak City Theme', artist: 'Johto', src: '/songs/song1.mp3' },
//   { id: 2, title: 'RSE Surf Theme', artist: 'Hoenn', src: '/songs/song2.mp3' },
// ]
const route = useRoute()
const router = useRouter()


const songs = ref([])
const currentSong = ref(null)
const isPlaying = ref(false)
const likedSongs = ref([])

function togglePlay(song) {
  if (currentSong.value?.id === song.id) {
    isPlaying.value = !isPlaying.value
  } else {
    currentSong.value = song
    isPlaying.value = true
  }
}

function playNext() {
  let nextSong
  
  if (route.path === '/liked') {
    const currentIndex = likedSongs.value.findIndex(song => song.id === currentSong.value.id)
    nextSong = likedSongs.value[currentIndex + 1]

  } else {
    const currentIndex = songs.value.findIndex(song => song.id === currentSong.value.id)

    nextSong = songs.value[currentIndex + 1]
  }


  if (nextSong) {
    togglePlay(nextSong)
  } else if (route.path !== '/liked') {
    fetchMoreSongs(currentSearchTerm.value)
  }
}

function playPrevious() {
  let previousSong
  
  if (route.path === '/liked') {
    const currentIndex = likedSongs.value.findIndex(song => song.id === currentSong.value.id)
    previousSong = likedSongs.value[currentIndex - 1]

  } else {
    const currentIndex = songs.value.findIndex(song => song.id === currentSong.value.id)

    previousSong = songs.value[currentIndex - 1]
  }


  if (previousSong) {
    togglePlay(previousSong)
  }
}


const currentSearchTerm = ref('tophits')
const songListRef = ref(null)
const fetchError = ref(false)
const noMoreNewSongsToFetch = ref(false)

async function fetchSongs(query) {
  router.push('/')
  currentSearchTerm.value = query

  
  try {
    const response = await fetch(`https://itunes.apple.com/search?term=${query}&media=music&limit=20`, { cache: 'no-store' })
    const data = await response.json()

    songs.value = data.results.map(track => ({
      id: track.trackId,
      title: track.trackName,
      artist: track.artistName,
      src: track.previewUrl,
      cover: track.artworkUrl100.replace('100x100', '600x600'),
      duration: track.trackTimeMillis / 1000
    }))

    fetchError.value = false
    noMoreNewSongsToFetch.value = false
    songListRef.value.scrollTop = 0 // scroll back to top
    // console.log(data)
    // console.log(songs.value)
    // offset.value = songs.value.length
  } catch (error) {
    console.warn('Error fetching more songs:', error)
    fetchError.value = true
    songs.value.length = 0
  }
}


const isLoadingMore = ref(false)

async function fetchMoreSongs(query) {
  if (isLoadingMore.value) return
  if (songs.value.length >= 100) return   // API can't return more than limit=200 (max limit)
  if (noMoreNewSongsToFetch.value) return  // no need to call api if it can't fetch new results

  const nextLimit = songs.value.length + 10

  isLoadingMore.value = true
  
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

    fetchError.value = false

    if (newSongs.length === 0) {
      noMoreNewSongsToFetch.value = true
      return  // if no new songs to fetch
    }

    songs.value = [...songs.value, ...newSongs]
    noMoreNewSongsToFetch.value = false
    // console.log(newSongs)

  } catch (error) {
      console.warn('Error fetching more songs:', error)
      fetchError.value = true
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


const isCurrentSongLiked = computed(()=>{
  if (!currentSong.value) return false

  return likedSongs.value.some(song => song?.id === currentSong.value.id)
})

function toggleLikeSong(song) {
  if (!song) return

  const index = likedSongs.value.findIndex(item=> item.id === song.id)
  // console.log(index)

  if (index !== -1) {
    likedSongs.value.splice(index, 1)
  } else {
    const songToStore = {
      id: song.id,
      title: song.title,
      artist: song.artist,
      src: song.src,
      cover: song.cover,
      duration: song.duration
    }
    
    // console.log(songToStore)
    likedSongs.value.push(songToStore)
  }

  localStorage.setItem('likedSongs', JSON.stringify(likedSongs.value))

}


onMounted(()=>{
    fetchSongs('tophits')

    const stored = localStorage.getItem('likedSongs')
    likedSongs.value = stored ? JSON.parse(stored) : []
    // console.log(likedSongs.value)
})

// Router related props
const routerProps = computed(()=> ({
  songs: songs.value,
  currentSong: currentSong.value,
  isPlaying: isPlaying.value,
  likedSongs: likedSongs.value,
  // isLiked: isCurrentSongLiked.value 
}))


</script>

<template>
  <div class="h-[100dvh] bg-background flex flex-col gap-6 p-4 text-text font-poppins">
    <!-- <div class="p-5"> -->
      <Searchbar @search="fetchSongs" />
    <!-- </div> -->

    
    <div class="bg-surface rounded-2xl overflow-hidden flex flex-col">
      <Navbar />
      <!-- BUG to fix later: Infinite scroll fails to trigger when initial content height is smaller than the container, because scroll bar doesn't appear (TO TRIGGER IT: try fetching 2 songs only)-->
      <div @scroll="handleScroll" ref="songListRef" class="p-3 flex flex-col overflow-y-auto flex-1 min-h-0 scroll-smooth
            [&::-webkit-scrollbar]:w-1.5
            [&::-webkit-scrollbar-track]:bg-transparent
            [&::-webkit-scrollbar-thumb]:bg-divider
            [&::-webkit-scrollbar-thumb]:rounded-full">


          <router-view v-bind="routerProps" @play="togglePlay" @like="toggleLikeSong"/>
          <!-- <SongItem v-for="song in songs" :key="song.id" :song="song"
            :isPlaying="isPlaying && currentSong?.id === song.id" @play="togglePlay"></SongItem> -->

          <div v-if="fetchError" class="text-center text-muted text-sm py-6">
            Couldn't load songs. Try searching for something else maybe.
          </div>
          <div v-else-if="songs.length===0 || isLoadingMore" class="text-center text-muted text-sm py-6">
            Loading...
          </div>
          <div v-else-if="route.path === '/liked' && likedSongs.length === 0" class="text-center text-muted text-sm py-6">
            No Liked Songs
          </div>
          

      </div>
    </div>

    <div class="bg-player rounded-2xl mt-auto">
      <Player :song="currentSong" :isPlaying="isPlaying" @toggle-play="togglePlay(currentSong)" @next="playNext" @prev="playPrevious" @ended="isPlaying=false; playNext()"  @like="toggleLikeSong" :is-liked="isCurrentSongLiked" />
    </div>
  </div>

</template>
