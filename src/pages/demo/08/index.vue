<script setup lang="ts">
import {
  createProgram,
  createShader,
  getWebGLContext,
  randomColor,
} from '@3dgl/utils'
import README from './README.md'
const canvas = ref<HTMLCanvasElement | null>(null)

onMounted(() => {
  if (canvas.value === null) throw new Error('canvas is null')
  const gl = getWebGLContext(canvas.value)

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
      gl_Position = vec4(position, 0.0, 1.0);
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
   * 初始化程序
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
   * 为顶点着色器中的 a_Screen_Size 传递 canvas 的宽高信息
   */
  gl.vertexAttrib2f(a_Screen_Size, canvas.value.width, canvas.value.height)

  /**
   * 创建坐标信息 buffer
   */
  const positionBuffer = gl.createBuffer()

  /**
   * 将当前 buffer 设置为 postionBuffer，接下来对 buffer 的操作都是针对 positionBuffer 了
   */
  gl.bindBuffer(gl.ARRAY_BUFFER, positionBuffer)

  /**
   * 设置 a_Position 变量读取 positionBuffer 缓冲区的方式。
   */
  gl.vertexAttribPointer(a_Position, 2, gl.FLOAT, false, 0, 0)

  gl.enableVertexAttribArray(a_Position)

  /**
   * 创建颜色信息 buffer
   */
  const colorBuffer = gl.createBuffer()

  /**
   * 将当前 buffer 设置为 colorBuffer
   */
  gl.bindBuffer(gl.ARRAY_BUFFER, colorBuffer)
  /**
   * 设置 a_Color 变量读取 colorBuffer 缓冲区的方式。
   */
  gl.vertexAttribPointer(a_Color, 4, gl.FLOAT, false, 0, 0)

  gl.enableVertexAttribArray(a_Color)

  gl.clearColor(0.0, 0.0, 0.0, 0.1)
  gl.clear(gl.COLOR_BUFFER_BIT)

  const colors: number[] = []
  const positions: number[] = []
  canvas.value.addEventListener('click', e => {
    const x = e.offsetX
    const y = e.offsetY
    positions.push(x, y)
    /**
     * 设置随机颜色
     */
    const { r, g, b, a } = randomColor()
    /**
     * 将随机颜色的 rgba 值添加到顶点颜色数组中
     */
    colors.push(r, g, b, a)

    /**
     * 顶点的数量是 3 的倍数时，执行绘制操作
     */
    if (positions.length % 6 === 0) {
      gl.bindBuffer(gl.ARRAY_BUFFER, positionBuffer)
      gl.bufferData(
        gl.ARRAY_BUFFER,
        new Float32Array(positions),
        gl.STATIC_DRAW,
      )

      gl.bindBuffer(gl.ARRAY_BUFFER, colorBuffer)
      gl.bufferData(gl.ARRAY_BUFFER, new Float32Array(colors), gl.STATIC_DRAW)

      /**
       * 绘制
       */
      gl.clear(gl.COLOR_BUFFER_BIT)
      gl.drawArrays(gl.TRIANGLES, 0, positions.length / 2)
    }
  })
})
</script>
<template>
  <ShowGL title="绘制渐变三角形（两个缓冲区）">
    <template #canvas>
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
