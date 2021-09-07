<script setup lang="ts">
import { createProgram, createShader, getWebGLContext } from '@3dgl/utils'
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
    attribute vec2 a_Uv;
    varying vec2 v_Uv;
    void main() {
      vec2 position = (a_Position / a_Screen_Size) * 2.0 - 1.0;
      position = position * vec2(1, 0, -1.0);
      gl_Position = vec4(position, 0, 1);
      v_Uv = a_Uv;
    }
  `

  /**
   * 定义片元着色器
   */
  const FRAG_SHADER_SOURCE = `
    precision mediump float;
    varying vec2 v_Uv;
    uniform sampler2D u_Texture;
    void main() {
      gl_FragColor = texture2D(u_Texture, vec2(v_Uv.x, v_Uv.y))
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
  const a_Uv = gl.getAttribLocation(program, 'a_Uv')
  /**
   * 找到着色器中的全局变量 u_Texture
   */
  const u_Texture = gl.getUniformLocation(program, 'u_Texture')

  /**
   * 顶点数据
   */
  const positions = [
    30, 30, 0, 0, // v0
    30, 370, 0, 1, // v1
    370, 370, 1, 1, // v2
    30, 30, 0, 0, // v0
    370, 370, 1, 1, // v2
    370, 30, 1, 0, // v3
  ]

  gl.vertexAttrib2f(a_Screen_Size, canvas.value.width, canvas.value.height)

  gl.enableVertexAttribArray(a_Position)
  gl.enableVertexAttribArray(a_Uv)

  /**
   * 创建缓冲区
   */
  const buffer = gl.createBuffer()
  /**
   * 绑定缓冲区为当前缓冲
   */
  gl.bindBuffer(gl.ARRAY_BUFFER, buffer)

  /**
   * 设置 a_Position 属性从缓冲区读取数据方式
   */
  gl.vertexAttribPointer(a_Position, 2, gl.FLOAT, false, 4 * (2 + 2), 0)
  /**
   * 设置 a_Uv 属性从缓冲区读取数据方式
   */
  gl.vertexAttribPointer(a_Uv, 2, gl.FLOAT, false, 4 * (2 + 2), 4 * 2)

  /**
   * 向缓冲区传入数据
   */
  gl.bufferData(gl.ARRAY_BUFFER, new Float32Array(positions), gl.STATIC_DRAW)

  /**
   * 清空绘图区
   */
  gl.clearColor(0.0, 0.0, 0.0, 0.1)

  const render = () => {
    gl.clear(gl.COLOR_BUFFER_BIT)
    /**
   * 绘制
   */
    gl.drawArrays(gl.TRIANGLES, 0, positions.length / 4)
  }

  render()

  const loadTexture = (gl: WebGLRenderingContext, src: string, texture: WebGLUniformLocation, callback: () => void) => {
    const img = new Image()
    img.src = src
    img.onload = callback
  }

  loadTexture(gl, '', u_Texture, () => {
    render()
  })
})
</script>
<template>
  <ShowGL title="纹理贴图">
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
