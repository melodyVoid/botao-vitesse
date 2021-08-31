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
   * 初始化程序
   */
  const { program } = createProgram(gl, vertexShader, fragShader)

  gl.useProgram(program)

  const a_Position = gl.getAttribLocation(program, 'a_Position')
  const a_Screen_Size = gl.getAttribLocation(program, 'a_Screen_Size')
  const u_Color = gl.getUniformLocation(program, 'u_Color')

  /**
   * 为顶点着色器中的 a_Screen_Size 传递 canvas 的宽高信息
   */
  gl.vertexAttrib2f(a_Screen_Size, canvas.value.width, canvas.value.height)

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
   * 点击 canvas 动态绘制线段
   */
  canvas.value.addEventListener('click', e => {
    const x = e.offsetX
    const y = e.offsetY
    // 两个坐标为一组
    positions.push(x, y)

    if (positions.length > 0) {
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
      gl.uniform4f(u_Color, 255, 0, 0, 1)
      gl.clear(gl.COLOR_BUFFER_BIT)
      /**
       * 绘制线段
       */
      gl.drawArrays(gl.LINES, 0, positions.length / 2)
    }
  })
})
</script>
<template>
  <ShowGL title="绘制线段（鼠标点击）">
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
