<script setup lang="ts">
import README from './README.md'
const canvas = ref<HTMLCanvasElement | null>(null)

onMounted(() => {
  if (canvas.value === null)
    throw new Error('canvas is null')

  const gl = canvas.value.getContext('webgl')

  if (gl === null)
    throw new Error('fail to get rendering context of WebGL')

  /**
   * 定义顶点着色器
   */
  const VERTEX_SHADER_SOURCE = `
    void main() {
      gl_Position = vec4(0.0, 0.0, 0.0, 1.0);
      gl_PointSize = 10.0;
    }
  `
  /**
   * 定义片元着色器
   */
  const FRAG_SHADER_SOURCE = `
    void main() {
      gl_FragColor = vec4(0.0, 0.0, 1.0, 1.0);
    }
  `
  /**
   * 创建着色器
   */
  const vertexShader = gl.createShader(gl.VERTEX_SHADER)
  const fragShader = gl.createShader(gl.FRAGMENT_SHADER)

  if (vertexShader === null || fragShader === null)
    throw new Error('创建着色器对象错误')

  gl.shaderSource(vertexShader, VERTEX_SHADER_SOURCE)
  gl.shaderSource(fragShader, FRAG_SHADER_SOURCE)

  gl.compileShader(vertexShader)
  gl.compileShader(fragShader)

  /**
   * 创建程序
   */
  const program = gl.createProgram()

  if (program === null)
    throw new Error('fail to create program')

  gl.attachShader(program, vertexShader)
  gl.attachShader(program, fragShader)

  gl.linkProgram(program)

  gl.useProgram(program)
  /**
   * 清空绘图区
   */
  gl.clearColor(0.0, 0.0, 0.0, 0.1)
  gl.clear(gl.COLOR_BUFFER_BIT)

  /**
   * 绘制
   */
  gl.drawArrays(gl.POINTS, 0, 1)
})
</script>
<template>
  <ShowGL title="绘制一个点" link="https://codesandbox.io/embed/02-hui-zhi-yi-ge-dian-cm4jn?fontsize=14&hidenavigation=1&theme=dark">
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
