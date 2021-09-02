# 利用三角形图元绘制矩形

## 基本三角形构建矩形

一个矩形其实可以由两个共线的三角形组成，即 V0, V1, V2, V3，其中 V0 -> V1 -> V2 代表三角形 A，V0 -> V2 -> V3 代表三角形 B。

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2018/9/11/165c77b134803832~tplv-t2oaga2asx-watermark.awebp)

**注意：** 组成三角形的顶点要按照一定的顺序绘制。默认情况下，WebGL 会认为**顶点顺序为逆时针时代表正面**，反之则是背面，区分正面、背面的目的在于，如果开启了背面剔除功能的话，背面是不会被绘制的。当我们绘制 3D 形体的时候，这个设置很重要。

## 着色器

### 顶点着色器

- a_Position
- a_Color
- a_Screen_Size
- v_Color

### 片元着色器

- v_Color

## JavaScript 部分

仍然从简单之处着手，绘制固定顶点的矩形。

首先准备组成矩形的三角形，每个三角形由三个顶点组成，两个矩形共需要六个顶点。

前两位代表位置，后四位代表颜色。

```js
/**
 * 顶点信息
 */
const positions = [
  30, 30, 255, 0, 0, 1, // v0
  30, 370, 255, 0, 0, 1, // v1
  370, 370, 255, 0, 0, 1, // v2
  30, 30, 0, 255, 0, 1, // v0
  370, 370, 0, 255, 0, 1, // v2
  370, 30, 0, 255, 0, 1, // v3
]
```

我们给两个三角形设置不同颜色，其中，V0->V1->V2 三角形设置为红色， VO->V2->V3 三角形设置为绿色。

## 完整代码

```js
import {
  createProgram,
  createShader,
  getWebGLContext,
  randomColor,
  createBuffer,
} from '@3dgl/utils'

/**
 * 获取 canvas 元素
 */
const canvas = document.getElementById('canvas')

const gl = getWebGLContext(canvas)

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
 * 为顶点着色器中的 a_Screen_Size 传递 canvas 的宽高信息
 */
gl.vertexAttrib2f(a_Screen_Size, canvas.value.width, canvas.value.height)

/**
 * 顶点信息
 */
const positions = [
  30, 30, 255, 0, 0, 1, // v0
	30, 370, 255, 0, 0, 1, // v1
	370, 370, 255, 0, 0, 1, // v2
	30, 30, 0, 255, 0, 1, // v0
	370, 370, 0, 255, 0, 1, // v2
	370, 30, 0, 255, 0, 1, // v3
]

/**
 * 创建 buffer
 */
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
gl.vertexAttribPointer(a_Color, 2, gl.FLOAT, false, 4 * (2 + 4), 2 * 4)

gl.enableVertexAttribArray(a_Position)
gl.enableVertexAttribArray(a_Color)

gl.clearColor(0.0, 0.0, 0.0, 0.1)
gl.clear(gl.COLOR_BUFFER_BIT)

gl.drawArrays(gl.TRIANGLES, 0, positions.length / 6)
```
