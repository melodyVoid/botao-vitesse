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
  <ShowGL title="清空绘图区" link="https://codesandbox.io/embed/01-qing-kong-hui-tu-qu-ln1li?fontsize=14&hidenavigation=1&theme=dark">
    <template #canvas>
      <canvas ref="canvas" width="400" height="400" class="w-80 sm:w-[400px]"></canvas>
    </template>
    <template #readme>
      <README />
    </template>
  </ShowGL>
</template>
<route lang="yaml">
meta:
  layout: empty
</route>
