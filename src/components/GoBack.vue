<script setup lang="ts">
const router = useRouter()

const goBack = () => router.push('/')
</script>
<template>
  <button class="text-gray-400 !outline-none" @click="goBack()">
    cd ..
  </button>
</template>
