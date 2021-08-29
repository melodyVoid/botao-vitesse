# 绘制多个点

上个例子中，我们只是绘制了一个点，而且点的位置和大小和颜色都是写死在着色器里面的，那么本例来聊一下怎么动态地绘制点。

我们要实现一个简单交互程序：在鼠标点击过的位置绘制一个点，点的颜色随机。

## 修改着色器程序

### 顶点着色器

```js
/**
 * 定义顶点着色器
 */
const VERTEX_SHADER_SOURCE = `
  precision mediump float;
  attribute vec2 a_Position;
  attribute vec2 a_Screen_Size;
  void main() {
    vec2 position = (a_Position / a_Screen_Size) * 2.0 - 1.0;
    position = position * vec2(1.0, -1.0);
    gl_Position = vec4(position, 0.0, 1.0);
    gl_PointSize = 10.0;
  }
`
```

解释：

`precision mediump float;` 设置浮点数精度为中等精度。

`attribute vec2 a_Position;` 接收点在 canvas 坐标系上的坐标 (x, y)。

`attribute vec2 a_Screen_Size;` 接收 canvas 的宽高尺寸

```js
vec2 position = (a_Position / a_Screen_Size) * 2.0 - 1.0;
position = position * vec2(1.0, -1.0);
gl_Position = vec4(position, 0.0, 1.0);
```

将屏幕坐标系转化为裁剪坐标（裁剪坐标系）

- `a_Position / a_Screen_Size` -> `[0, 1]`
- 乘以 2 -> `[0, 2]`
- 减 1 -> `[-1, 1]`

我们在顶点着色器中定义两个 attribute 变量： a_Position 和 a_Screen_Size，a_Position 接收 canvas 坐标系下的点击坐标。 vec2 代表存储两个浮点数变量的容器，本例不涉及**深度计算**，所以我们只接收顶点的 x 和 y 坐标。

### 片元着色器

```js
/**
 * 定义片元着色器
 */
const FRAG_SHADER_SOURCE = `
  precision mediump float;
  uniform vec4 u_Color;
  void main() {
    vec4 color = u_Color / vec4(255, 255, 255, 1);
    gl_FragColor = color;
  }
`
```

解释：

`uniform vec4 u_Color;` 接收 JavaScript 传递过来的颜色值 RGBA 。

`vec4 color = u_Color / vec4(255, 255, 255, 1);` 将普通的颜色表示转化为 WebGL 需要的表示方式，即将 `[0-255]` 转化到 `[0,1]` 之间。

片元着色器定义了一个全局变量 (被 uniform 修饰的变量) ，用来接收 JavaScript 传递过来的随机颜色。

## 获取着色器中的变量

我们可以通过 `gl.getAttribLocation` 和 `gl.getUniformLocation` 函数来获取着色器中的变量。

```js
/**
 * 找到顶点着色器中的变量 a_Position
 */
const a_Position = gl.getAttribLocation(program, 'a_Position')
/**
 * 找到顶点着色器中的变量 a_Screen_Size
 */
const a_Screen_Size = gl.getAttribLocation(program, 'a_Screen_Size')
/**
 * 找到片元着色器中的变量 u_Color
 */
const u_Color = gl.getUniformLocation(program, 'u_Color')
```

## 向变量中赋值

为 attribute 变量赋值可使用 `gl.vertexAttrib2f` 函数，因为我们定义 `a_Position` 和 `a_Screen_Size` 时用的是 vec2，所以我们在赋值的时候需要用 `gl.vertexAttrib2f`。这里的 vec2 和 2f 是对应的。

```js
/**
 * 为顶点着色器中的 a_Screen_Size 传递 canvas 的宽高信息
 */
gl.vertexAttrib2f(a_Screen_Size, canvas.width, canvas.height)
```

```js
/**
 * 为顶点着色器中的 a_Position 传递顶点坐标。
 */
gl.vertexAttrib2f(a_Position, x, y)
```

在为片元着色器中的 u_Color 传递颜色信息时，我们用到的是 `gl.uniform4f`。同理，因为我们定义 u_Color 时用的是 vec4，所以在赋值时要用 4f。

```js
/**
 * 为片元着色器中的 u_Color 传递颜色信息
 */
gl.uniform4f(u_Color, color.r, color.g, color.b, color.a)
```

## 动态绘制点的逻辑

1. 声明一个数组变量 points，存储点击位置的坐标。
2. 绑定 canvas 的点击事件。
3. 触发点击操作时，把点击坐标添加到数组 points 中，并将颜色设置为随机色。
4. 遍历每个点执行 `drawArrays(gl.Points, 0, 1)` 绘制操作。

## 完整代码

```js
import {
  getWebGLContext,
  createShader,
  createProgram,
  randomColor,
} from '@3dgl/utils'
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
  attribute vec2 a_Screen_Size;
  void main() {
    
    vec2 position = (a_Position / a_Screen_Size) * 2.0 - 1.0;
    position = position * vec2(1.0, -1.0);
    gl_Position = vec4(position, 0.0, 1.0);
    gl_PointSize = 10.0;
  }
`
/**
 * 定义片元着色器
 */
const FRAG_SHADER_SOURCE = `
  precision mediump float;
  uniform vec4 u_Color;
  void main() {
    vec4 color = u_Color / vec4(255, 255, 255, 1);
    gl_FragColor = color;
  }
`

/**
 * 创建着色器
 */
const vertexShader = createShader(gl, gl.VERTEX_SHADER, VERTEX_SHADER_SOURCE)
const fragShader = createShader(gl, gl.FRAGMENT_SHADER, FRAG_SHADER_SOURCE)

/**
 * 创建 Program
 */
const { program } = createProgram(gl, vertexShader, fragShader)

/**
 * 使用程序
 */
gl.useProgram(program)

/**
 * 清空背景
 */
gl.clearColor(0.0, 0.0, 0.0, 0.1)
gl.clear(gl.COLOR_BUFFER_BIT)

/**
 * 找到顶点着色器中的变量 a_Position
 */
const a_Position = gl.getAttribLocation(program, 'a_Position')
/**
 * 找到顶点着色器中的变量 a_Screen_Size
 */
const a_Screen_Size = gl.getAttribLocation(program, 'a_Screen_Size')
/**
 * 找到片元着色器中的变量 u_Color
 */
const u_Color = gl.getUniformLocation(program, 'u_Color')

/**
 * 为顶点着色器中的 a_Screen_Size 传递 canvas 的宽高信息
 */
gl.vertexAttrib2f(a_Screen_Size, canvas.width, canvas.height)

/**
 * 存储点击位置的数组
 */
const points = []

canvas.addEventListener('click', e => {
  const x = e.offsetX
  const y = e.offsetY

  /**
   * 生成随机颜色
   */
  const color = randomColor()
  points.push({ x, y, color })
  // 绘制前清空屏幕
  gl.clear(gl.COLOR_BUFFER_BIT)
  // 循环绘制
  points.forEach(({ x, y, color }) => {
    gl.vertexAttrib2f(a_Position, x, y)
    gl.uniform4f(u_Color, color.r, color.g, color.b, color.a)
    gl.drawArrays(gl.POINTS, 0, 1)
  })
})
```

## 总结

### 定义着色器变量

- attribue 变量：只能在**顶点着色器**中定义。

- uniform 变量：既可以在**顶点着色器**中定义，也可以在**片元着色器**中定义。

- varing 变量：它用来从**顶点着色器**中往**片元着色器**传递数据。使用它我们可以在顶点着色器中声明一个变量并对其赋值，经过插值处理后，在片元着色器中取出插值后的值来使用。

### JavaScript 向着色器中传递数据

- getAttribLocation：找到着色器中的 attribute 变量地址。

- getUniformLocation：找到着色器中的 uniform 变量地址。

- vertexAttrib2f：给 attribute 变量传递两个浮点数。

- uniform4f：给 uniform 变量传递四个浮点数。
