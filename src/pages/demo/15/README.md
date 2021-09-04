# 绘制圆形

绘制圆形：将圆形分割成以圆心为共同顶点的若干个三角形，三角形数越多，圆形越平滑。

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2018/9/21/165fb3037eeca21b~tplv-t2oaga2asx-watermark.awebp)

如上图所示，我们将圆形划分成 12 个三角形，13 个顶点，我们需要计算每个顶点的坐标，我们定义一个生成圆顶点的函数：

- x：圆心的 x 坐标
- y：圆心的 y 坐标
- radius：半径
- n：三角形的数量

```js
function createCircleVertex(x, y, radius, n) {
	const positions = [x, y, 255, 0, 0, 1]
	for (let i = 0; i <= n; i++) {
		const angle = (i * 2 * Math.PI) / n
		const vertex = [x + radius * Math.sin(angle), y + radius * Math.cos(angle), 255, 0, 0, 1]
		positions.push(...vertex)
	}
	return positions
}
```

然后将圆划分成 12 个三角形

```js
const positions = createCircleVertex(200, 200, 150, 12)
```

效果如下：

![](https://files.catbox.moe/eqf906.png)

棱角还是很明显的，下面我们将圆切分成 50 个三角形试试

```js
const positions = createCircleVertex(200, 200, 150, 50)
```

效果如下：

![](https://files.catbox.moe/zna5ez.png)

可以看出， 50 个三角形组成的圆更加自然一些，三角形面数越多，画出的图形越自然越平滑，但是我们也不能无限划分，毕竟三角形的数量越多，顶点数量相应的变多，内存占用会变大。在绘制规则图形的时候，我们需要在图形显示效果与顶点数量之间做一个权衡。

[![Edit 15-绘制圆形](https://codesandbox.io/static/img/play-codesandbox.svg)](https://codesandbox.io/s/15-hui-zhi-yuan-xing-4uldh?fontsize=14&hidenavigation=1&theme=dark)

## 完整代码

```ts
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
 * 传递 canvas 宽高信息
 */
gl.vertexAttrib2f(a_Screen_Size, canvas.width, canvas.height)

function createCircleVertex(x: number, y: number, radius: number, n: number) {
	const positions = [x, y, 255, 0, 0, 1]
	for (let i = 0; i <= n; i++) {
		const angle = (i * 2 * Math.PI) / n
		const vertex = [
			x + radius * Math.sin(angle),
			y + radius * Math.cos(angle),
			0,
			0,
			0,
			1,
		]
		positions.push(...vertex)
	}
	return positions
}

/**
 * 生成顶点信息
 */
const positions = createCircleVertex(200, 200, 150, 50)

/**
 * 创建缓冲区
 */
const buffer = gl.createBuffer()
/**
 * 绑定缓冲区为当前缓冲区
 */
gl.bindBuffer(gl.ARRAY_BUFFER, buffer)

/**
 * 向缓冲区传递数据
 */
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
 * 用三角扇的图元进行绘制
 */
gl.drawArrays(gl.TRIANGLE_FAN, 0, positions.length / 6)
```