# 绘制三角形

## 三角形图元分类

三角形图元又可以分为三类：

1. 基本三角形（TRIANGLES），绘制三角形的数量 = 顶点数 / 3
2. 三角带（TRIANGLE_STRIP），绘制三角形的数量 = 顶点数 - 2
3. 三角扇（TRIANGLE_FAN），绘制三角形的数量 = 顶点数 - 2

## 着色器

### 顶点着色器

我们这里沿用之前的顶点着色器，因为不涉及深度计算，所以我们使用 vec2，每个顶点坐标之传入 x 和 y 即可。

```js
/**
 * 定义顶点着色器
 */
const VERTEX_SHADER_SOURCE = `
	precision mediump float;
	attribute vec2 a_Position;
	void main() {
		gl_Position = vec4(a_Position, 0, 1);
	}
`
```

### 片元着色器

片元着色器我们还是在着色器里进行换算，JS 传进来是普通的 RGBA 颜色值。

```js
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
```

## 初始化着色器和程序

这部分和之前一样，使用 [@3dgl/utils](https://www.npmjs.com/package/@3dgl/utils) 提供的工具函数方法来写。

## 准备数据

### 着色器变量

找到顶点着色器中的变量 a_Position

```js
const a_Position = gl.getAttribLocation(program, 'a_Position')
```

找到片元着色器中的变量 u_Color

```js
const u_Color = gl.getUniformLocation(program, 'u_Color')
```

### 顶点数据

这里我们使用 Float32Array 来定义顶点数据，使用类型化数组与 WebGL 进行通信，性能好。

```js
const positions = new Float32Array([0.0, 0.0, -1.0, -1.0, 1.0, -1.0])
```

## 使用缓冲区

使用缓冲区的步骤：

1. 创建缓冲区（`gl.createBuffer()`）；
2. 绑定缓冲区（`gl.bindBuffer()`）；
3. 将数据写入缓冲区对象（`gl.bufferData()`）；
4. 将缓冲区对象分配给一个 attribute 变量（`gl.vertexAttribPointer()`）；
5. 开启 attribute 变量（`gl.enableVertexAttribArray()`）。

```js
/**
 * 创建缓冲区
 */
const buffer = gl.createBuffer()
/**
 * 绑定缓冲区
 */
gl.bindBuffer(gl.ARRAY_BUFFER, buffer)

/**
 * 向当前缓冲区写入数据
 */
gl.bufferData(gl.ARRAY_BUFFER, positions, gl.STATIC_DRAW)

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
```

## 清空绘图区

```js
gl.clearColor(0.0, 0.0, 0.0, 0.1)
gl.clear(gl.COLOR_BUFFER_BIT)
```

## 设置颜色并绘制

```js
/**
 * 设置颜色
 */
gl.uniform4f(u_Color, 15, 118, 110, 1)
/**
 * 画三角形
 */
gl.drawArrays(gl.TRIANGLES, 0, 3)
```

## 完整代码

```js
/**
 * 获取 canvas 元素
 */
const canvas = document.getElementById('canvas')

/**
 * 获取 webgl 绘制上下文
 */
const gl = getWebGLContext(canvas)

/**
 * 定义顶点着色器
 */
const VERTEX_SHADER_SOURCE = `
	precision mediump float;
	attribute vec2 a_Position;
	void main() {
		gl_Position = vec4(a_Position, 0, 1);
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
 * 创建程序
 */
const { program } = createProgram(gl, vertexShader, fragShader)

/**
 * 使用 program
 */
gl.useProgram(program)

/**
 * 清空绘图区
 */
gl.clearColor(0.0, 0.0, 0.0, 0.1)
gl.clear(gl.COLOR_BUFFER_BIT)

/**
 * 找到顶点着色器中的变量 a_Position
 */
const a_Position = gl.getAttribLocation(program, 'a_Position')
/**
 * 找到片元着色器中的变量 u_Color
 */
const u_Color = gl.getUniformLocation(program, 'u_Color')

/**
 * 顶点数据
 */
const positions = new Float32Array([0.0, 0.0, -1.0, -1.0, 1.0, -1.0])

/**
 * 创建缓冲区
 */
const buffer = gl.createBuffer()
/**
 * 绑定缓冲区
 */
gl.bindBuffer(gl.ARRAY_BUFFER, buffer)

/**
 * 向当前缓冲区写入数据
 */
gl.bufferData(gl.ARRAY_BUFFER, positions, gl.STATIC_DRAW)

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
 * 设置颜色
 */
gl.uniform4f(u_Color, 15, 118, 110, 1)
/**
 * 画三角形
 */
gl.drawArrays(gl.TRIANGLES, 0, 3)
```
