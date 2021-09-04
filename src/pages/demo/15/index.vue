<script setup lang="ts">
import { createProgram, createShader, getWebGLContext } from '@3dgl/utils'
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
   * 传递 canvas 宽高信息
   */
  gl.vertexAttrib2f(a_Screen_Size, canvas.value.width, canvas.value.height)

  function createCircleVertex(x: number, y: number, radius: number, n: number) {
    const positions = [x, y, 255, 0, 0, 1]
    for (let i = 0; i <= n; i++) {
      const angle = (i * 2 * Math.PI) / n
      const vertex = [
        x + radius * Math.sin(angle),
        y + radius * Math.cos(angle),
        0,
        0,
        0,
        1,
      ]
      positions.push(...vertex)
    }
    return positions
  }

  /**
   * 生成顶点信息
   */
  const positions = createCircleVertex(200, 200, 150, 50)

  /**
   * 创建缓冲区
   */
  const buffer = gl.createBuffer()
  /**
   * 绑定缓冲区为当前缓冲区
   */
  gl.bindBuffer(gl.ARRAY_BUFFER, buffer)

  /**
   * 向缓冲区传递数据
   */
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

  /**
   * 用三角扇的图元进行绘制
   */
  gl.drawArrays(gl.TRIANGLE_FAN, 0, positions.length / 6)
})
</script>
<template>
  <ShowGL title="绘制圆形" link="https://codesandbox.io/embed/15-hui-zhi-yuan-xing-4uldh?fontsize=14&hidenavigation=1&theme=dark">
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
