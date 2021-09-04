# 利用索引绘制矩形

在上个例子绘制矩形的时候，实际上我们只需要 4 个顶点即可，但是却定义了 6 个。

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

每个顶点占据 4 * 6 = 24 个字节，绘制一个简单的矩形我们就浪费了 24 * 2 = 48 字节的空间，那真正的 WebGL 应用都是由成百上千个，甚至几十万、上百万个顶点组成，这个时候，重复的顶点信息所造成的内存浪费就不容小觑了。

如何改进呢？

WebGL 除了提供 gl.drawArrays 按顶点绘制的方式以外，还提供了一种按照顶点索引进行绘制的方法：gl.drawElements，使用这种方式，可以避免重复定义顶点，进而节省存储空间。我们看下 gl.drawElements 的使用方法，详细解释参见[MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/WebGLRenderingContext/drawElements)。

```js
gl.drawElements(mode, count, type, offset);
```

- mode：指定绘制图元的类型，是画点，还是画线，或者是画三角形。
- count：指定绘制图形的顶点个数。
- type：指定索引缓冲区中的值的类型,常用的两个值：`gl.UNSIGNED_BYTE` 和`gl.UNSIGNED_SHORT`，前者为无符号 8 位整数值，后者为无符号 16 位整数。
- offset：指定索引数组中开始绘制的位置，以字节为单位。

举例来说：

```js
gl.drawElements(gl.TRIANGLES, 3, gl.UNSIGNED_BYTE, 0);
```

这段代码的意思是：采用三角形图元进行绘制，共绘制 3 个顶点，顶点索引类型是 `gl.UNSIGNED_BYTE`，从顶点索引数组的开始位置绘制。

# 使用 drawElements 绘制矩形

## 着色器

着色器部分不做改动

## JavaScript 部分

我们的 JavaScript 部分要有所改变了，采用索引绘制方式，我们除了准备存储顶点信息的数组，还要准备存储顶点索引的数组。

```js
/**
 * 存储顶点信息的数组
 */
const positions = [
	30, 30, 255, 0, 0, 1, // v0
	30, 370, 255, 0, 0, 1, // v1
	370, 370, 255, 0, 0, 1, // v2
	370, 30, 0, 255, 0, 1, // v3
]
/**
 * 存储顶点索引的数组
 */
const indices = [
	0, 1, 2, // 第一个三角形
	0, 2, 3 // 第二个三角形
]
```

除了多准备一个数组容器存储顶点索引以外，我们还需要将索引传递给 GPU，所以，仍然需要创建一个索引 buffer。

```js
const indicesBuffer = gl.createBuffer()
```
按照惯例，创建完 buffer，我们需要绑定，这里要和 ARRAY_BUFFER 区分开来，索引 buffer 的绑定点是gl.ELEMENT_ARRAY_BUFFER。

```js
gl.bindBuffer(gl.ELEMENT_ARRAY_BUFFER, indicesBuffer)
```

接下来，我们就可以往 indicesBuffer 中传入顶点索引了：

```js
gl.bufferData(gl.ELEMENT_ARRAY_BUFFER, new Uint16Array(indices), gl.STATIC_DRAW)
```

之后执行绘制操作：

```js
gl.drawElements(gl.TRIANGLES, 6, gl.UNSIGNED_SHORT, 0)
```

矩形能够绘制出来，但是颜色和我们之前的矩形有些不同，第二个三角形从红到绿渐变。

仔细回顾一下，我们用 drawArrays 进行绘制的时候，使用了六个顶点，每个三角形的顶点颜色一致，所以两个三角形的颜色都是单一的。 当采用 drawElements 方法进行绘制的时候，使用了四个顶点，第二个三角形的两个顶点 V0、V2 是红色的，第三个顶点 V3 是绿色的，所以造成了从 V0、V2 向 V3 的红绿渐变。

如果我们必须要实现两个不同颜色的单色三角形，还是应该用六个顶点来绘制，这时，使用 drawArrays 的方式更优一些。毕竟，不用创建索引数组和索引缓冲。

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
gl.vertexAttribPointer(a_Color, 4, gl.FLOAT, false, 4 * (2 + 4), 2 * 4)

gl.enableVertexAttribArray(a_Position)
gl.enableVertexAttribArray(a_Color)

/**
 * 存储顶点索引的数组
 */
const indices = [
	0, 1, 2, // 第一个三角形
	0, 2, 3, // 第二个三角形
]

/**
 * 创建索引缓冲区
 */
const indicesBuffer = gl.createBuffer()
/**
 * 绑定索引缓冲区
 */
gl.bindBuffer(gl.ELEMENT_ARRAY_BUFFER, indicesBuffer)
/**
 * 向索引缓冲区传递索引数据
 */
gl.bufferData(gl.ELEMENT_ARRAY_BUFFER, new Uint16Array(indices), gl.STATIC_DRAW)
/**
 * 清空绘制区
 */
gl.clearColor(0.0, 0.0, 0.0, 0.1)
gl.clear(gl.COLOR_BUFFER_BIT)
/**
 * 绘制
 */
gl.drawElements(gl.TRIANGLES, indices.length, gl.UNSIGNED_SHORT, 0)

```