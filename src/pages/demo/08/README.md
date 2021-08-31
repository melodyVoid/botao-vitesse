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