<script setup lang="ts">
import README from './README.md'
const canvas = ref<HTMLCanvasElement | null>(null)

onMounted(() => {
  if (canvas.value === null)
    throw new Error('canvas is null')

  const gl = canvas.value.getContext('webgl')

  if (gl === null)
    throw new Error('fail to get rendering context of WebGL')

  gl.clearColor(0.0, 0.0, 0.0, 0.1)
  gl.clear(gl.COLOR_BUFFER_BIT)
})
</script>
<template>
  <div class="block lg:flex justify-center gap-6 lg:h-[calc(100vh-70px)] overflow-hidden ">
    <div class="py-6 flex flex-col items-center justify-center">
      <canvas ref="canvas" width="400" height="400"></canvas>
      <span class="text-gray-500 mt-4">清空绘图区</span>
      <GoBack />
    </div>
    <div class="box-border flex items-center overflow-y-scroll my-6 overflow-hidden lg:h-[calc(100vh-3rem-70px)]">
      <README />
    </div>
  </div>
</template>
<route lang="yaml">
meta:
  layout: empty
</route>
