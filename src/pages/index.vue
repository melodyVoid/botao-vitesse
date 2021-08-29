<script setup lang="ts">

const demos = [
  { title: '01-第一个 WebGL 程序：清空绘图区', path: '/demo/01' },
]

</script>

<template>
  <div>
    <p class="text-4xl font-thin italic flex items-center justify-center">
      <simple-icons:webgl class="inline-block text-6xl mr-4" /> demos
    </p>

    <ul>
      <li v-for="(item, index) of demos" :key="index" class="text-gray-400 hover:text-gray-600">
        <router-link :to="item.path">
          {{ item.title }}
        </router-link>
      </li>
    </ul>
  </div>
</template>

<route lang="yaml">
meta:
  layout: default
</route>
