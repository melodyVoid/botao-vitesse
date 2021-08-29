<script setup lang="ts">
import README from './README.md'
const canvas = ref<HTMLCanvasElement | null>(null)

onMounted(() => {
  if (canvas.value === null)
    throw new Error('canvas is null')

  const gl = canvas.value.getContext('webgl')

  if (gl === null)
    throw new Error('fail to get rendering context of WebGL')
})
</script>
<template>
  <ShowGL title="优化绘制一个点">
    <template #canvas>
      <canvas ref="canvas" width="400" height="400" class="w-80 sm:w-[400px]"></canvas>
    </template>
    <template #readme>
      <README />
    </template>
  </ShowGL>
</template>
