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
    }
  `

  /**
   * 定义片元着色器
   */
  const FRAG_SHADER_SOURCE = `
    precision mediump float;
    uniform vec4 u_Color;
    void main() {
      gl_FragColor = u_Color / vec4(255, 255, 255, 1);
    }
  `

  /**
   * 初始化着色器
   */
  const vertexShader = createShader(gl, gl.VERTEX_SHADER, VERTEX_SHADER_SOURCE)
  const fragShader = createShader(gl, gl.FRAGMENT_SHADER, FRAG_SHADER_SOURCE)

  /**
   * 创建程序
   */
  const { program } = createProgram(gl, vertexShader, fragShader)

  /**
   * 使用 program
   */
  gl.useProgram(program)

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
   * 顶点数据
   */
  const positions: number[] = []

  /**
   * 创建缓冲区
   */
  const buffer = gl.createBuffer()
  /**
   * 绑定缓冲区
   */
  gl.bindBuffer(gl.ARRAY_BUFFER, buffer)

  // 每次取两个数据
  const size = 2
  // 每个数据的类型是32位浮点型
  const type = gl.FLOAT
  // 不需要归一化数据
  const normalize = false
  // 每次迭代运行需要移动数据数 * 每个数据所占内存 到下一个数据开始点。
  const stride = 0
  // 从缓冲起始位置开始读取
  const offset = 0
  // 将 a_Position 变量获取数据的缓冲区指向当前绑定的 buffer。
  gl.vertexAttribPointer(a_Position, size, type, normalize, stride, offset)

  /**
   * 开启 attribute 变量
   */
  gl.enableVertexAttribArray(a_Position)

  /**
   * 清空绘图区
   */
  gl.clearColor(0.0, 0.0, 0.0, 0.1)
  gl.clear(gl.COLOR_BUFFER_BIT)

  /**
   * 点击 canvas
   */
  canvas.value.addEventListener('click', e => {
    const x = e.offsetX
    const y = e.offsetY
    // 两个坐标为一组
    positions.push(x, y)

    /**
     * 顶点信息为 6 个数据即 3 个顶点时执行绘制操作，因为三角形由三个顶点组成。
     */
    if (positions.length % 6 === 0) {
      /**
       * 向当前缓冲区写入数据
       */
      gl.bufferData(
        gl.ARRAY_BUFFER,
        new Float32Array(positions),
        gl.STATIC_DRAW,
      )
      /**
       * 设置颜色
       */
      const { r, g, b, a } = randomColor()
      gl.uniform4f(u_Color, r, g, b, a)

      gl.clear(gl.COLOR_BUFFER_BIT)
      /**
       * 画三角形
       */
      gl.drawArrays(gl.TRIANGLES, 0, positions.length / 2)
    }
  })
})
</script>
<template>
  <ShowGL title="动态绘制三角形（鼠标点击三下）">
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
