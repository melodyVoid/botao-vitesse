<script setup lang="ts">
import { createProgram, createShader, getWebGLContext, randomColor } from '@3dgl/utils'
import README from './README.md'

function createRingVertex(x: number, y: number, innerRadius: number, outerRadius: number, n: number) {
  const positions = []
  let color = randomColor()
  for (let i = 0; i <= n; i++) {
    if (i % (n / 4) === 0)
      color = randomColor()

    const angle = (i * Math.PI * 2) / n
    positions.push(
      x + innerRadius * Math.sin(angle),
      y + innerRadius * Math.cos(angle),
      color.r,
      color.g,
      color.b,
      1,
    )
    positions.push(
      x + outerRadius * Math.sin(angle),
      y + outerRadius * Math.cos(angle),
      color.r,
      color.g,
      color.b,
      1,
    )
  }
  const indices = []
  for (let i = 0; i < n; i++) {
    const p0 = i * 2
    const p1 = i * 2 + 1
    let p2 = (i + 1) * 2 + 1
    let p3 = (i + 1) * 2
    if (i === n - 1) {
      p2 = 1
      p3 = 0
    }
    indices.push(p0, p1, p2, p2, p3, p0)
  }
  return { positions, indices }
}

const canvas = ref<HTMLCanvasElement>(null!)

onMounted(() => {
  const gl = getWebGLContext(canvas.value)

  /*
   * 顶点着色器
   */
  const VERTEX_SHADER_SOURCE = `
    // 浮点数设置为中等精度
    precision mediump float;
    // 接收 Javascript 传递过来的坐标（x, y) 
    attribute vec2 a_Position;
    // 接收 canvas 尺寸
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
      vec4 color = v_Color / vec4(255, 255, 255, 1);
      gl_FragColor = color;
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
  const a_Color = gl.getAttribLocation(program, 'a_Color')
  const a_Screen_Size = gl.getAttribLocation(program, 'a_Screen_Size')

  gl.enableVertexAttribArray(a_Position)
  gl.enableVertexAttribArray(a_Color)

  /**
   * 传递 canvas 宽高信息
   */
  gl.vertexAttrib2f(a_Screen_Size, canvas.value.width, canvas.value.height)

  /**
   * 定义组成矩形的两个三角形，共计四个顶点
   * 每个顶点包含2个坐标分量(x, y)和4个颜色分量(r, g, b, a)
   * 其中 V0,V1,V2代表左下角三角形，V1,V2,V3代表右上角三角形。
   *
   * positions: 顶点数量
   * indices: 定义绘制索引数组
  */
  const { positions, indices } = createRingVertex(200, 200, 80, 150, 1000)

  /***
   *
   * ☆☆☆☆☆ ☆☆☆☆☆ ☆☆☆☆☆ ☆☆☆☆☆ ☆☆☆☆☆ ☆☆☆☆☆
   * 以下 buffer 的操作需要在绑定 buffer 之后进行：
   *gl.bufferData：传递数据。
    ● gl.vertexAttribPointer：设置属性读取 buffer 的方式。
    ☆☆☆☆☆ ☆☆☆☆☆ ☆☆☆☆☆ ☆☆☆☆☆ ☆☆☆☆☆ ☆☆☆☆☆
   *
   */

  // 创建缓冲区
  const buffer = gl.createBuffer()
  // 绑定缓冲区为当前缓冲
  gl.bindBuffer(gl.ARRAY_BUFFER, buffer)
  // 向缓冲区传递数据
  gl.bufferData(gl.ARRAY_BUFFER, new Float32Array(positions), gl.STATIC_DRAW)
  // 设置 a_Position 属性从缓冲区读取数据的方式
  gl.vertexAttribPointer(a_Position, 2, gl.FLOAT, false, 24, 0)
  // 设置 a_Color 属性从缓冲区读取数据方式
  gl.vertexAttribPointer(a_Color, 4, gl.FLOAT, false, 24, 8)

  // 创建索引缓冲区
  const indicesBuffer = gl.createBuffer()
  // 绑定索引缓冲区
  gl.bindBuffer(gl.ELEMENT_ARRAY_BUFFER, indicesBuffer)
  // 向索引缓冲区传递索引数据
  gl.bufferData(gl.ELEMENT_ARRAY_BUFFER, new Uint16Array(indices), gl.STATIC_DRAW)

  /**
   * 清空屏幕
   */
  gl.clearColor(0.0, 0.0, 0.0, 0.1)
  gl.clear(gl.COLOR_BUFFER_BIT)

  const render = () => {
    gl.clear(gl.COLOR_BUFFER_BIT)
    gl.drawElements(gl.TRIANGLES, indices.length, gl.UNSIGNED_SHORT, 0)
  }

  render()
})

</script>

<template>
  <ShowGL title="绘制环形" link="https://codesandbox.io/s/19-hui-zhi-huan-xing-qqh1n">
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
