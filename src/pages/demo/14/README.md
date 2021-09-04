# 顶点顺序

其实不管是使用三角扇还是基本三角形，又或者是三角带绘制的时候，一定要保证顶点顺序是逆时针。如果三角形的顶点顺序不是逆时针，在开启背面剔除功能后，不是逆时针顺序的三角形是不会被绘制的。

我们不妨试一下，改变顶点顺序，将他们之间的关系从逆时针改为顺时针：

![](https://files.catbox.moe/esxj3i.png)

还是上个例子的图，这回我们顺时针绘制，还是以 v1 为公共顶点，

- 第一个三角形：v1 -> v0 -> v2
- 第二个三角形：v1 -> v2 -> v3

```js
const positions = [
	370, 370, 255, 0, 0, 1, // v1
	30, 370, 255, 0, 0, 1, // v0
	30, 30, 255, 0, 0, 1, // v2
	370, 30, 0, 255, 0, 1, // v3
]
```

这样看起来好像没有问题，也绘制成功了。但是如果开启了**背面剔除**功能，就会发现什么都没有了。

```js
gl.enable(gl.CULL_FACE)
```

当然，我们也可以更改面的显示方式，默认显示正面，我们可以通过如下方式，剔除正面，只显示背面：

```js
gl.cullFace(gl.FRONT)
```

[![Edit 14-顶点顺序](https://codesandbox.io/static/img/play-codesandbox.svg)](https://codesandbox.io/s/14-ding-dian-shun-xu-hn50r?fontsize=14&hidenavigation=1&theme=dark)

## 完整代码

```js
import { createProgram, createShader, getWebGLContext } from '@3dgl/utils'

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
	30, 370, 255, 0, 0, 1, // v0
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

const render = () => {
  gl.clear(gl.COLOR_BUFFER_BIT)

  /**
   * 使用三角扇绘制
   */
  gl.drawArrays(gl.TRIANGLE_FAN, 0, positions.length / 6)
}

render()

const disableButton = document.getElementById('button1')
const enableButton = document.getElementById('button2')

disableButton!.addEventListener('click', () => {
  gl.disable(gl.CULL_FACE)
  render()
})

enableButton!.addEventListener('click', () => {
  gl.enable(gl.CULL_FACE)
  render()
})
```