<script setup lang="ts">
import { createProgram, createShader, getWebGLContext } from '@3dgl/utils'
import README from './README.md'
const canvas = ref<HTMLCanvasElement | null>(null)
const glRef = ref<WebGLRenderingContext | null>(null)

/**
 * 顶点信息
 */
const positions = [
  370,
  370,
  255,
  0,
  0,
  1, // v1
  30,
  370,
  255,
  0,
  0,
  1, // v0
  30,
  30,
  255,
  0,
  0,
  1, // v2
  370,
  30,
  0,
  255,
  0,
  1, // v3
]
/**
 * 背面剔除状态
 */
enum CULL_FACE {
  ENABLE,
  DISABLE,
}

const render = (state: CULL_FACE) => {
  const gl = glRef.value
  if (gl === null) throw new Error('can not get rendering context of WebGL')

  if (state === CULL_FACE.ENABLE) {
    /**
     * 开启背面剔除
     */
    gl.enable(gl.CULL_FACE)
    // eslint-disable-next-line brace-style
  } else {
    /**
     * 关闭背面剔除
     */
    gl.disable(gl.CULL_FACE)
  }

  gl.clear(gl.COLOR_BUFFER_BIT)
  /**
   * 使用三角扇绘制
   */

  gl.drawArrays(gl.TRIANGLE_FAN, 0, positions.length / 6)
}
onMounted(() => {
  if (canvas.value === null) throw new Error('canvas is null')

  const gl = getWebGLContext(canvas.value)
  glRef.value = gl

  /**
   * 定义顶点着色器
   */
  const VERTEX_SHADER_SOURCE = `
    precision mediump float;
    attribute vec2 a_Position;
    attribute vec2 a_Screen_Size;
    attribute vec4 a_Color;
    varying vec4 v_Color;
    void main() {
      vec2 position = (a_Position / a_Screen_Size) * 2.0 - 1.0;
      position = position * vec2(1.0, -1.0);
      gl_Position = vec4(position, 0, 1);
      v_Color = a_Color;
    }
  `

  /**
   * 定义片元着色器
   */
  const FRAG_SHADER_SOURCE = `
    precision mediump float;
    varying vec4 v_Color;
    void main() {
      gl_FragColor = v_Color / vec4(255, 255, 255, 1);
    }
  `

  /**
   * 初始化着色器
   */
  const vertexShader = createShader(gl, gl.VERTEX_SHADER, VERTEX_SHADER_SOURCE)
  const fragShader = createShader(gl, gl.FRAGMENT_SHADER, FRAG_SHADER_SOURCE)

  /**
   * 初始化 program
   */
  const { program } = createProgram(gl, vertexShader, fragShader)

  gl.useProgram(program)

  /**
   * 准备数据
   */
  const a_Position = gl.getAttribLocation(program, 'a_Position')
  const a_Screen_Size = gl.getAttribLocation(program, 'a_Screen_Size')
  const a_Color = gl.getAttribLocation(program, 'a_Color')

  /**
   * 传递 canvas 宽高信息
   */
  gl.vertexAttrib2f(a_Screen_Size, canvas.value.width, canvas.value.height)

  const buffer = gl.createBuffer()
  gl.bindBuffer(gl.ARRAY_BUFFER, buffer)

  gl.bufferData(gl.ARRAY_BUFFER, new Float32Array(positions), gl.STATIC_DRAW)

  /**
   * 设置 a_Position 属性从缓冲区读取数据方式
   */
  gl.vertexAttribPointer(a_Position, 2, gl.FLOAT, false, 4 * (2 + 4), 0)
  /**
   * 设置 a_Color 属性从缓冲区读取数据方式
   */
  gl.vertexAttribPointer(a_Color, 4, gl.FLOAT, false, 4 * (2 + 4), 2 * 4)

  gl.enableVertexAttribArray(a_Position)
  gl.enableVertexAttribArray(a_Color)

  gl.clearColor(0.0, 0.0, 0.0, 0.1)
  gl.clear(gl.COLOR_BUFFER_BIT)

  // /**
  //  * 开启背面剔除
  //  */
  // gl.enable(gl.CULL_FACE)

  /**
   * 使用三角扇绘制
   */

  gl.drawArrays(gl.TRIANGLE_FAN, 0, positions.length / 6)
})
</script>
<template>
  <ShowGL title="顶点顺序" link="https://codesandbox.io/embed/14-ding-dian-shun-xu-hn50r?fontsize=14&hidenavigation=1&theme=dark">
    <template #canvas>
      <div class="flex justify-start gap-3">
        <button
          class="btn my-3 text-sm mt-8 !outline-transparent"
          @click="render(CULL_FACE.DISABLE)"
        >
          关闭背面剔除
        </button>
        <button
          class="btn my-3 text-sm mt-8 !outline-none"
          @click="render(CULL_FACE.ENABLE)"
        >
          开启背面剔除
        </button>
      </div>
      <canvas
        ref="canvas"
        width="400"
        height="400"
        class="w-80 sm:w-[400px]"
      ></canvas>
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
