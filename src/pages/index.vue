<script setup lang="ts">

const demos = [
  { title: '01-第一个 WebGL 程序：清空绘图区', path: '/demo/01' },
  { title: '02-绘制一个点', path: '/demo/02' },
  { title: '03-绘制一个点（优化代码）', path: '/demo/03' },
  { title: '04-绘制多个点', path: '/demo/04' },
  { title: '05-绘制三角形', path: '/demo/05' },
  { title: '06-动态绘制三角形', path: '/demo/06' },
  { title: '07-绘制线段', path: '/demo/07' },
  { title: '08-绘制渐变三角形（两个缓冲区）', path: '/demo/08' },
]

</script>

<template>
  <div>
    <p class="text-4xl w-80 mx-auto font-thin italic flex items-center justify-center">
      <simple-icons:webgl class="inline-block text-6xl mr-4" /> demos
    </p>

    <ul class="text-left mx-auto w-80">
      <li v-for="(item, index) of demos" :key="index" class="my-1 text-gray-400 hover:text-gray-600">
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
