<script setup lang="ts">
import {
  getWebGLContext,
  createShader,
  createProgram,
  randomColor,
} from '@3dgl/utils'
import README from './README.md'
const canvas = ref<HTMLCanvasElement | null>(null)

onMounted(() => {
  if (canvas.value === null) throw new Error('canvas is null')

  const gl = getWebGLContext(canvas.value as HTMLCanvasElement)
  /**
   * 定义顶点着色器
   */
  const VERTEX_SHADER_SOURCE = `
    precision mediump float;
    attribute vec2 a_Position;
    attribute vec2 a_Screen_Size;
    void main() {
      vec2 position = (a_Position / a_Screen_Size) * 2.0 - 1.0;
      position = position * vec2(1.0, -1.0);
      gl_Position = vec4(position, 0.0, 1.0);
      gl_PointSize = 10.0;
    }
  `

  /**
   * 定义片元着色器
   */
  const FRAG_SHADER_SOURCE = `
    precision mediump float;
    uniform vec4 u_Color;
    void main() {
      vec4 color = u_Color / vec4(255, 255, 255, 1);
      gl_FragColor = color;
    }
  `

  /**
   * 创建着色器
   */
  const vertexShader = createShader(gl, gl.VERTEX_SHADER, VERTEX_SHADER_SOURCE)
  const fragShader = createShader(gl, gl.FRAGMENT_SHADER, FRAG_SHADER_SOURCE)

  /**
   * 创建 Program
   */
  const { program } = createProgram(gl, vertexShader, fragShader)

  /**
   * 使用程序
   */
  gl.useProgram(program)

  /**
   * 清空背景
   */
  gl.clearColor(0.0, 0.0, 0.0, 0.1)
  gl.clear(gl.COLOR_BUFFER_BIT)

  /**
   * 找到顶点着色器中的变量 a_Position
   */
  const a_Position = gl.getAttribLocation(program, 'a_Position')
  /**
   * 找到顶点着色器中的变量 a_Screen_Size
   */
  const a_Screen_Size = gl.getAttribLocation(program, 'a_Screen_Size')
  /**
   * 找到片元着色器中的变量 u_Color
   */
  const u_Color = gl.getUniformLocation(program, 'u_Color')

  /**
   * 为顶点着色器中的 a_Screen_Size 传递 canvas 的宽高信息
   */
  gl.vertexAttrib2f(a_Screen_Size, canvas.value.width, canvas.value.height)

  /**
   * 存储点击位置的数组
   */
  const points: Array<{
    x: number
    y: number
    color: { r: number; g: number; b: number; a: number }
  }> = []

  canvas.value.addEventListener('click', e => {
    const x = e.offsetX
    const y = e.offsetY
    const color = randomColor()
    points.push({ x, y, color })
    // 绘制前清空屏幕
    gl.clear(gl.COLOR_BUFFER_BIT)
    // 循环绘制
    points.forEach(({ x, y, color }) => {
      gl.vertexAttrib2f(a_Position, x, y)
      gl.uniform4f(u_Color, color.r, color.g, color.b, color.a)
      gl.drawArrays(gl.POINTS, 0, 1)
    })
  })
})
</script>
<template>
  <ShowGL title="绘制多个点（用鼠标点击）">
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
