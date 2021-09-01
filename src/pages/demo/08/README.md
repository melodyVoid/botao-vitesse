# 绘制渐变三角形

之前的例子是向着色器传递**坐标**这一种数据，本例通过绘制渐变三角形，了解一下通过缓冲区向着色器传递多种数据。

绘制单色三角形时我们通过在片元着色器中定义一个 `uniform` 变量，接收 JavaScript 传递过来的颜色值来实现。

渐变三角形的处理和单色三角形有何不同呢？

渐变三角形颜色不单一，在顶点与顶点之间进行颜色的渐变过渡，这就要求我们的顶点信息**除了包含坐标，还要包含颜色**。这样在顶点着色器之后，GPU 根据每个顶点的颜色对顶点与顶点之间的颜色进行**插值**，自动填补顶点之间像素的颜色，于是形成了渐变三角形。

那既然我们需要为每个顶点传递坐标信息和颜色信息，因此需要在顶点着色器中额外增加一个 attribute 变量 `a_Color`，用来接收顶点的颜色，同时还需要在顶点着色器和片元着色器中定义一个 varying 类型的变量 `v_Color`，用来传递顶点颜色信息。

## 着色器

### 顶点着色器

依然从顶点着色器开始，顶点着色器新增一个 attribute 变量，用来接收顶点颜色。

```js
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
```

### 片元着色器

片元着色器新增一个 varying 变量 `v_Color`，用来接收插值后的颜色。

```js
const FRAG_SHADER_SOURCE = `
	precision mediump float;
	varying vec4 v_Color;
	void main() {
		gl_FragColor = v_Color / vec4(255, 255, 255, 1);
	}
`
```

## JavaScript 传递数据

用缓冲区向着色器传递数据有两种方式：

1. 利用一个缓冲区传递多种数据；
2. 利用多个缓冲区传递多个数据；

之前在绘制三角形的例子中，我们给顶点着色器传递的只是坐标信息，并且只用了一个 buffer。

本例我们除了传递顶点坐标数据，还要传递顶点颜色。我们创建两个 `buffer`，其中一个 `buffer` 传递坐标，一个 `buffer` 传递颜色。

创建两个 `buffer`：

`a_Position` 和 `positionBuffer` 绑定

`a_Color` 和 `colorBuffer` 绑定

然后设置各自读取 `buffer` 的方式。

**注意：**程序中如果有多个 `buffer` 的时候，在切换 `buffer` 进行操作时，一定要通过调用 `gl.bindBuffer` 将要操作的 `buffer` 绑定到 `gl.ARRAY_BUFFER` 上，这样才能正确地操作 `buffer` 。我们可以将 `bindBuffer` 理解为一个状态机，`bindBuffer` 之后的对 `buffer` 的一些操作，都是基于最近一次绑定的 `buffer` 来进行的。

以下 `buffer` 的操作需要在绑定 `buffer` 之后进行：

- `gl.bufferData`：传递数据。
- `gl.vertexAttribPointer`：设置属性读取 `buffer` 的方式。

```js
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
```

由于这两部分代码基本一样，我们将它们抽离到工具函数库中。

```ts
/**
 * 创建 buffer
 */
export interface VertexAttribPointerOptions {
  /**
   * 每次取几个数据
   */
  size: number
  /**
   * 数据类型，一般为 gl.Float
   */
  type?: number
  /**
   * 是否需要归一化
   */
  normalized?: boolean
  /**
   * 每次迭代运行需要移动数据数 * 每个数据所占内存 到下一个数据开始点。
   */
  stride?: number
  /**
   * 从缓冲位置偏移量
   */
  offset?: number
}
export const createBuffer = (
  gl: WebGLRenderingContext,
  attribute: number,
  vertexAttribPointer: VertexAttribPointerOptions,
) => {
  const {
    size,
    type = gl.FLOAT,
    normalized = false,
    stride = 0,
    offset = 0,
  } = vertexAttribPointer

  const buffer = gl.createBuffer()
  gl.bindBuffer(gl.ARRAY_BUFFER, buffer)

  gl.vertexAttribPointer(attribute, size, type, normalized, stride, offset)
  gl.enableVertexAttribArray(attribute)

  return buffer
}
```

接下来使用就简单了，可以直接调用 `createBuffer`

```js
import { createBuffer } from '@3dgl/utils'

/**
 * 创建坐标信息 buffer
 */
const positionBuffer = createBuffer(gl, a_Position, { size: 2 })

/**
 * 创建颜色信息 buffer
 */
const colorBuffer = createBuffer(gl, a_Color, { size: 4 })
```

## 添加点击事件

```js
const colors = []
const positions = []
canvas.addEventListener('click', e => {
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
    gl.bufferData(gl.ARRAY_BUFFER, new Float32Array(positions), gl.STATIC_DRAW)

    gl.bindBuffer(gl.ARRAY_BUFFER, colorBuffer)
    gl.bufferData(gl.ARRAY_BUFFER, new Float32Array(colors), gl.STATIC_DRAW)

    /**
     * 绘制
     */
    gl.clear(gl.COLOR_BUFFER_BIT)
    gl.drawArrays(gl.TRIANGLES, 0, positions.length / 2)
  }
})
```

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
gl.vertexAttrib2f(a_Screen_Size, canvas.width, canvas.height)

/**
 * 创建坐标信息 buffer
 */
const positionBuffer = createBuffer(gl, a_Position, { size: 2 })

/**
 * 创建颜色信息 buffer
 */
const colorBuffer = createBuffer(gl, a_Color, { size: 4 })

gl.clearColor(0.0, 0.0, 0.0, 0.1)
gl.clear(gl.COLOR_BUFFER_BIT)

const colors = []
const positions = []
canvas.addEventListener('click', e => {
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
    gl.bufferData(gl.ARRAY_BUFFER, new Float32Array(positions), gl.STATIC_DRAW)

    gl.bindBuffer(gl.ARRAY_BUFFER, colorBuffer)
    gl.bufferData(gl.ARRAY_BUFFER, new Float32Array(colors), gl.STATIC_DRAW)

    /**
     * 绘制
     */
    gl.clear(gl.COLOR_BUFFER_BIT)
    gl.drawArrays(gl.TRIANGLES, 0, positions.length / 2)
  }
})
```
