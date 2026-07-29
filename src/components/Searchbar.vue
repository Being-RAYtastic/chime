<script setup>
import { ref } from 'vue'

const searchTerm = ref('')
const emit = defineEmits(['search'])

function handleSearch() {
    if (searchTerm.value.trim() !== '') emit('search', searchTerm.value)
}

</script>



<template>
    <div class="flex items-center gap-3 w-full">
        <div class="w-10 mr-2 flex">
            <img src="/src/assets/icon.png" class="" />
        </div>
        <div class="flex-1">
            <input type="text" v-model="searchTerm" @keyup.enter="handleSearch" placeholder="Search"
                class="w-full bg-surface text-text rounded-2xl px-4 py-2 outline-none focus:ring-2 focus:ring-accent" />
        </div>
        <div>
            <button @click="handleSearch" class="bg-accent text-background rounded-2xl px-4 py-2 font-medium cursor-pointer
                transition-colors
                active:scale-95">
                Search
            </button>
        </div>
    </div>
</template>