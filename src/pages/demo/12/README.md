# 使用三角带绘制矩形

上个例子中我们使用了基本三角形绘制矩形，接下来我们使用三角带绘制矩形。

三角带绘制特点是前后两个三角形是共线的，并且我们知道顶点数量与绘制的三角形的数量之间的关系是：

```js
顶点数或索引数 = 三角形数量 + 2
```

仍然以绘制矩形为目标，如果采用基本三角形进行绘制的话，需要准备六个顶点，即两个三角形。那如果采用三角带进行绘制的话，利用三角带的特性，我们实际需要的顶点数为 2 + 2 = 4，即矩形的四个顶点位置。

**注意：顶点顺序不能乱**

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2018/9/12/165cc89ab4b4a667~tplv-t2oaga2asx-watermark.awebp)

由上图可以看出，绘制三角带图元的时候，v0 -> v1 -> v2 组成第一个三角形，v2 -> v1 -> v3 组成第二个三角形。

三角带与基本三角形绘制在代码上的区别有两点：

1. 顶点数组的数据不同。
2. drawArrays 的第一个参数代表的图元类型不同。
	- 基本三角形：TRIANGLES。
	- 三角带：TRIANGLE_STRIP。
	- 三角扇：TRIANGLE_FAN。

## 关键代码

顶点数组

```js
const positions = [
	30, 370, 255, 0, 0, 1, // v0
	370, 370, 255, 0, 0, 1, // v1
	30, 30, 255, 0, 0, 1, // v2
	370, 30, 0, 255, 0, 1, // v3
]
```

绘制方法

```js
gl.drawArrays(gl.TRIANGLE_STRIP, 0, 4)
```

[![Edit 使用三角带绘制矩形](https://codesandbox.io/static/img/play-codesandbox.svg)](https://codesandbox.io/s/shi-yong-san-jiao-dai-hui-zhi-ju-xing-kwc5j?autoresize=1&fontsize=14&hidenavigation=1&theme=dark)

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
* 存储顶点信息的数组
*/
const positions = [
	30, 370, 255, 0, 0, 1, // v0
	370, 370, 255, 0, 0, 1, // v1
	30, 30, 255, 0, 0, 1, // v2
	370, 30, 0, 255, 0, 1, // v3
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
 * 使用三角带绘制
 */
gl.drawArrays(gl.TRIANGLE_STRIP, 0, 4)
```