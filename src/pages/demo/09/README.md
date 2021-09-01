# 绘制渐变三角形（一个缓冲区）

上个例子中我们通过两个缓冲区 `positionBuffer` 和 `colorBuffer` 实现了渐变三角形的绘制。注意点就是在切换 buffer 时需要先 `bindBuffer`。

接下来我们用一个缓冲区来实现同样的功能。

着色器部分的代码无需改动，改动的主要是 JavaScript 部分的程序。

## 创建缓冲区

```js
/**
 * 创建 buffer
 */
const buffer = gl.createBuffer()

gl.bindBuffer(gl.ARRAY_BUFFER, buffer)
```

创建完 `buffer`，接下来设置读取 `buffer` 的方式，我们有两个属性 `a_Position` 和 `a_Color`，由于我们只有一个 `buffer`，该 `buffer` 中既存储坐标信息，又存储颜色信息，所以两个属性需要读取同一个 `buffer`。

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2018/9/10/165c2e171e604257~tplv-t2oaga2asx-watermark.awebp)

我们可以看到，一个顶点信息占用 6 个元素，前两个元素代表坐标信息，后四个元素代表颜色信息，所以在下面设置属性读取 buffer 方式时，a_Color 和 a_Position 的设置会有不同：

- a_Position：坐标信息占用 2 个元素，故 size 设置为 2。 坐标信息是从第一个元素开始读取，偏移值为 0 ，所以 offset 设置为 0.

- a_Color：由于 color 信息占用 4 个元素，所以 size 设置为 4 。 color 信息是在坐标信息之后，偏移两个元素所占的字节（2 \* 4 = 8）。所以，offset 设置为 8。

- stride：代表一个顶点信息所占用的字节数，我们的示例，一个顶点占用 6 个元素，每个元素占用 4 字节，所以，stride = 4 \* 6 = 24 个字节。

```js
// 4-每个数字占 4 个字节，（2 + 4）的意思是：每个顶点由 2 个坐标信息和 4 个颜色信息组成
gl.vertexAttribPointer(a_Position, 2, gl.FLOAT, false, 4 * (2 + 4), 0)
// 2 * 4 是偏移量，前 8 个字节是坐标信息，后面的是颜色信息
gl.vertexAttribPointer(a_Color, 4, gl.FLOAT, false, 4 * (2 + 4), 2 * 4)
```

## 点击事件

canvas 的点击事件也有所不同，一个顶点占用 6 个元素，三个顶点组成一个三角形，所以我们的 vertices 的元素数量必须是 18 的整数倍，才能组成一个三角形：

```js
const vertices = []
canvas.addEventListener('click', e => {
  const x = e.offsetX
  const y = e.offsetY
  // 将位置信息添加到顶点数组中
  vertices.push(x, y)

  const { r, g, b, a } = randomColor()
  // 将颜色信息添加到顶点数组中
  vertices.push(r, g, b, a)

  // 顶点是 18（每个顶点 6 个值，三个顶点为一个三角形） 的倍数时，执行绘制操作
  if (vertices.length % 18 === 0) {
    gl.bufferData(gl.ARRAY_BUFFER, new Float32Array(vertices), gl.STATIC_DRAW)

    /**
     * 绘制
     */
    gl.clear(gl.COLOR_BUFFER_BIT)
    gl.drawArrays(gl.TRIANGLES, 0, vertices.length / 6)
  }
})
```

实现效果和上面操作多缓冲区的方式一样，但是单缓冲区不仅减少了缓冲区的数量，而且减少了传递数据的次数以及复杂度。

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
const buffer = gl.createBuffer()

gl.bindBuffer(gl.ARRAY_BUFFER, buffer)
// 4-每个数字占 4 个字节，（2 + 4）的意思是：每个顶点由 2 个坐标信息和 4 个颜色信息组成
gl.vertexAttribPointer(a_Position, 2, gl.FLOAT, false, 4 * (2 + 4), 0)
// 2 * 4 是偏移量，前 8 个字节是坐标信息，后面的是颜色信息
gl.vertexAttribPointer(a_Color, 4, gl.FLOAT, false, 4 * (2 + 4), 2 * 4)

gl.enableVertexAttribArray(a_Position)
gl.enableVertexAttribArray(a_Color)

gl.clearColor(0.0, 0.0, 0.0, 0.1)
gl.clear(gl.COLOR_BUFFER_BIT)

const vertices = []
canvas.addEventListener('click', e => {
  const x = e.offsetX
  const y = e.offsetY
  // 将位置信息添加到顶点数组中
  vertices.push(x, y)

  const { r, g, b, a } = randomColor()
  // 将颜色信息添加到顶点数组中
  vertices.push(r, g, b, a)

  // 顶点是 18（每个顶点 6 个值，三个顶点为一个三角形） 的倍数时，执行绘制操作
  if (vertices.length % 18 === 0) {
    gl.bufferData(gl.ARRAY_BUFFER, new Float32Array(vertices), gl.STATIC_DRAW)

    /**
     * 绘制
     */
    gl.clear(gl.COLOR_BUFFER_BIT)
    gl.drawArrays(gl.TRIANGLES, 0, vertices.length / 6)
  }
})
```


