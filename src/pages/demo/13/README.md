# 使用三角扇绘制矩形

这个例子我们通过三角扇绘制矩形。

三角扇是围绕找第一个顶点作为公共顶点绘制三角形的，并且使用 `三角扇` 绘制出来的三角形的数量和顶点数量之间的关系和 `三角带` 一样：

```js
顶点数或者索引数 = 三角形数量 + 2
```

我们看下三角扇绘制矩形时的顶点分布以及顺序：


![](https://files.catbox.moe/esxj3i.png)

我们绘制两个具有共同顶点的三角形，顶点数量为 4 个。

以 v1 为共同顶点，然后**逆时针画**。

- 第一个三角形：v1 -> v3 -> v2

- 第二个三角形：v1 -> v2 -> v0

顶点信息数组为：

```js
const positions = [
	370, 370, 255, 0, 0, 1, // v1
	370, 30, 0, 255, 0, 1, // v3
	30, 30, 255, 0, 0, 1, // v2
	30, 370, 255, 0, 0, 1, // v0
]
```

然后绘制方式改为三角扇：

```js
gl.drawArrays(gl.TRIANGLE_FAN, 0, positions.length / 6)
```

效果如左边的示例。

[![Edit 13-使用三角扇绘制矩形](https://codesandbox.io/static/img/play-codesandbox.svg)](https://codesandbox.io/s/13-shi-yong-san-jiao-shan-hui-zhi-ju-xing-xsylj?fontsize=14&hidenavigation=1&theme=dark)

## 完整代码

```js
import {
  createProgram,
  createShader,
  getWebGLContext,
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
gl.vertexAttrib2f(a_Screen_Size, canvas.width, canvas.height)

/**
 * 顶点信息
 */
const positions = [
	370, 370, 255, 0, 0, 1, // v1
	370, 30, 0, 255, 0, 1, // v3
	30, 30, 255, 0, 0, 1, // v2
	30, 370, 255, 0, 0, 1, // v0
]

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
gl.vertexAttribPointer(a_Color, 4, gl.FLOAT, false, 4 * (2 + 4), 2 * 4)

gl.enableVertexAttribArray(a_Position)
gl.enableVertexAttribArray(a_Color)

gl.clearColor(0.0, 0.0, 0.0, 0.1)
gl.clear(gl.COLOR_BUFFER_BIT)

/**
 * 使用三角扇绘制
 */
gl.drawArrays(gl.TRIANGLE_FAN, 0, positions.length / 6)
```