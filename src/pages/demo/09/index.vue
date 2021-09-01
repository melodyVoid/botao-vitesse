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
   * 顶点着色器
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
   * 创建 buffer
   */
  const buffer = gl.createBuffer()

  gl.bindBuffer(gl.ARRAY_BUFFER, buffer)
  // 4-每个数字占 4 个字节，（2 + 4）的意思是：每个顶点由 2 个坐标信息和 4 个颜色信息组成
  gl.vertexAttribPointer(a_Position, 2, gl.FLOAT, false, 4 * (2 + 4), 0)
  // 2 * 4 是偏移量，前 8 个字节是坐标信息，后面的是颜色信息
  gl.vertexAttribPointer(a_Color, 4, gl.FLOAT, false, 4 * (2 + 4), 2 * 4)

  gl.enableVertexAttribArray(a_Position)
  gl.enableVertexAttribArray(a_Color)

  gl.clearColor(0.0, 0.0, 0.0, 0.1)
  gl.clear(gl.COLOR_BUFFER_BIT)

  const vertices: number[] = []
  canvas.value.addEventListener('click', e => {
    const x = e.offsetX
    const y = e.offsetY
    // 将位置信息添加到顶点数组中
    vertices.push(x, y)

    const { r, g, b, a } = randomColor()
    // 将颜色信息添加到顶点数组中
    vertices.push(r, g, b, a)

    // 顶点是 18（每个顶点 6 个值，三个顶点为一个三角形） 的倍数时，执行绘制操作
    if (vertices.length % 18 === 0) {
      gl.bufferData(gl.ARRAY_BUFFER, new Float32Array(vertices), gl.STATIC_DRAW)

      /**
       * 绘制
       */
      gl.clear(gl.COLOR_BUFFER_BIT)
      gl.drawArrays(gl.TRIANGLES, 0, vertices.length / 6)
    }
  })
})
</script>
<template>
  <ShowGL title="绘制渐变三角形（一个缓冲区）">
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
