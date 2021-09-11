# 绘制正方体

## WebGL 坐标系

3D 形体的顶点坐标系需要包含深度信息 z 轴坐标。所以我们先了解一下 WebGL 坐标系的概念。

WebGL 采用了左手坐标系，x 轴向右为正，y 轴向上为正，z 轴沿着屏幕往里为正。

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2018/9/29/16624dfadfcfef91~tplv-t2oaga2asx-watermark.awebp)

WebGL 是遵循右手坐标系，但仅仅是遵循，是期望大家遵守的规范。其实 WebGL 内部（裁剪坐标系）是基于左手坐标系的，z 轴沿屏幕向里为正方向。

WebGL 坐标系 X、Y、Z 三个坐标分量的的范围是 [-1，1]，即一个边长为 2 的正方体，原点在正方体中心。这点在之前的章节有介绍，我们也称这个坐标系为**标准设备坐标系**，简称 **NDC 坐标系**。

前面我们经常在顶点着色器中使用内置属性 `gl_Position`，并且在为 `gl_Position` 赋值之前做了一些坐标系转换（屏幕坐标系转换到裁剪坐标系）操作。

> 为了理解 gl_Position 接收坐标前所做的变换目的，这就需要理解 `gl_Position` 接收什么样的坐标。

`gl_Position` 接收一个 4 维浮点向量，该向量代表的是**裁剪坐标系**的坐标。

`gl_Position` 接收的坐标范围是顶点在裁剪坐标系中的坐标。

裁剪坐标系中的坐标通常由四个分量表示：(x, y, z, w)。请注意，w 分量代表**齐次坐标分量**，在之前的例子中，w 都是设置成 1 ，这样做的目的是让裁剪坐标系和 NDC 坐标系就保持一致，省去裁剪坐标到 NDC 坐标的转换过程。

`gl_Position` 接收到裁剪坐标之后，顶点着色器会对坐标进行透视除法，透视除法的公式是 (x/w, y/w, z/w, w/w) ，透视除法过后，顶点在裁剪坐标系中的坐标就会变成 NDC 坐标系中的坐标，各个坐标的取值范围将被限制在 [-1，1] 之间，如果某个坐标超出这个范围，将会被 GPU 丢弃。

> 透视除法这个步骤是顶点着色器程序黑盒执行的，对开发者来说是透明的，无法通过编程手段干预。但是我们需要明白有这么一个过程存在。

在前面的例子中，我们给出的顶点坐标都是基于**屏幕坐标系**，然后在顶点着色器中对顶点作简单转换处理，转变成 NDC 坐标。

```js
const VERTEX_SHADER_SOURCE = `
  precision mediump float;
  attribute vec2 a_Position;
  attribute vec2 a_Screen_Size;
  void main() {
    vec2 position = (a_Position / a_Screen_Size) * 2.0 - 1.0;
    position = position * vec2(1.0, -1.0);
    gl_Position = vec4(position, 0, 1);
  }
`
```

本节我们不着重讲解坐标系变换，而是为了讲解物体如何由三角形组成，所以会忽略裁剪坐标系之前的一些坐标变换，在 JavaScript 中直接采用**裁剪坐标系坐标**来表示顶点位置。

## 用三角形构建正方体

一个只包含坐标信息的立方体实际上是由 6 个正方形，每个正方形由两个三角形组成，每个三角形由三个顶点组成，所以一个立方体由 6 个正方形 * 2 个三角形 * 3 个顶点 = 36 个顶点组成，但是这 36个顶点中有很多是重复的，我们很容易发现：一个纯色立方体实际上由 6 个矩形面，或者 8 个不重复的顶点组成。

**注意：**顶点的重复与否，不只取决于顶点的坐标信息一致，还取决于该顶点所包含的其他信息是否一致。比如顶点纹理坐标 uv、顶点法线，顶点颜色等。**一旦有一个信息不同，就必须用两个顶点来表示。**

仍然以矩形举例，每个顶点只包含**坐标**和**颜色**两类信息。如果我们的矩形是纯色的，假设是红色。

```js
// 顶点信息
const positions = [
  30, 30, 1, 0, 0, 1, // v0
  30, 370, 1, 0, 0, 1, // v1
  370, 370, 1, 0, 0, 1, // v2
  30, 30, 1, 0, 0, 1, // v0,
  370, 370, 1, 0, 0, 1, // v2
  370, 30, 1, 0, 0, 1, // v3
]
```
很明显，v0 和 v2 这两个顶点坐标和颜色完全一致，所以，该顶点是重复的，我们可以忽略重复的顶点。

同样地，还是这样一个矩形，每个顶点还是只包含坐标和颜色两类信息，我们想实现一个渐变矩形，从 v0 -> v1v2 为红绿渐变，从v1v2 -> v3 为黄蓝渐变。 如下图所示：

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2018/9/28/1661ee227cedb437~tplv-t2oaga2asx-watermark.awebp)

顶点数组

```js
const positions = [
  30, 30, 1, 0, 0, 1, // v0 红色
  30, 370, 0, 1, 0, 1, // v1 绿色
  370, 370, 0, 1, 0, 1, // v2 绿色
  30, 30, 1, 1, 0, 1, // v4 黄色
  370, 370, 1, 1, 0, 1, // v5 黄色
  370, 30, 0, 0, 1, 1, // v3 蓝色
]
```

可以看出，虽然 v0 和 v4，v2 和 v5 的顶点坐标一致，但是顶点颜色不一样，所以我们只能把他们当成不同的顶点处理，否则达不到想要的效果。

> 一定要理解重复顶点的定义：两个顶点必须是所有信息一致，才可以称之为重复顶点。

## 彩色立方体

为了在视觉层面区分出立方体的各个面，接下来我们绘制一个彩色立方体。

立方体是 3 维形体，所以它们的顶点坐标需要从 2 维扩展成 3 维，除了 x、y 坐标，还需要深度值： z 轴坐标。

### 代码调整

本节代码组织上和之前章节有所不同，主要有以下两点：

- 顶点属性不再使用一个 buffer 混合存储，改为每个属性对应一个 buffer，便于维护。

- 顶点坐标我们不再使用屏幕坐标系，而是采用 NDC 坐标系。如果使用屏幕坐标系，会涉及到相对复杂的坐标系变换，大家可能不容易理解。

我们还是之前的套路：

- 定义顶点
- 传递数据
- 执行绘制

首先定义顶点，由于立方体包含六个面，每个面采用同一个颜色，所以我们需要定义 6 个矩形面 * 4 个顶点 = 24 个不重复的顶点。

```js
// 正方体 8 个顶点的坐标信息
const zeroX = 0.5
const zeroY = 0.5
const zeroZ = 0.5

const positions = [
  [-zeroX, -zeroY, zeroZ], // v0
  [zeroX, -zeroY, zeroZ], // v1
  [zeroX, zeroY, zeroZ], // v2
  [-zeroX, zeroY, zeroZ], // v3
  [-zeroX, -zeroY, -zeroZ], // v4
  [-zeroX, zeroY, -zeroZ], // v5
  [zeroX, zeroY, -zeroZ], // v6
  [zeroX, -zeroY, -zeroZ], // v7
]
```
接下来定义六个面包含的顶点索引：

```js
const CUBE_FACE_INDICES = [
  [0, 1, 2, 3], // 前面
  [4, 5, 6, 7], // 后面
  [0, 3, 5, 4], // 左面
  [1, 7, 6, 2], // 右面
  [3, 2, 6, 5], // 上面
  [0, 4, 7, 1], // 下面
]
```

定义六个面的颜色信息

```js
const FACE_COLORS = [
  [1, 0, 0, 1], // 前面，红色
  [0, 1, 0, 1], // 后面，绿色
  [0, 0, 1, 1], // 左面，蓝色
  [1, 1, 0, 1], // 右面，黄色
  [1, 0, 1, 1], // 上面，品色
  [0, 1, 1, 1], // 下面，青色
]
```

有了顶点坐标和颜色信息，接下来我们写一个方法生成立方体的顶点属性。该方法接收三个参数：宽度、高度、深度，返回一个包含组成立方体的顶点坐标、颜色、索引的对象。

```js
function createCube(width, height, depth) {
  const zeroX = width / 2
  const zeroY = height / 2
  const zeroZ = depth / 2

  const cornerPositions = [
    [-zeroX, -zeroY, -zeroZ],
    [zeroX, -zeroY, -zeroZ],
    [zeroX, zeroY, -zeroZ],
    [-zeroX, zeroY, -zeroZ],
    [-zeroX, -zeroY, zeroZ],
    [-zeroX, zeroY, zeroZ],
    [zeroX, zeroY, zeroZ],
    [zeroX, -zeroY, zeroZ]
  ]

  const colorInput = [
    [255, 0, 0, 1],
    [0, 255, 0, 1],
    [0, 0, 255, 1],
    [255, 255, 0, 1],
    [0, 255, 255, 1],
    [255, 0, 255, 1]
  ]

  let colors = []
  let positions = []
  let indices = []

  for (let f = 0; f < 6; f++) {
    const faceIndices = CUBE_FACE_INDICES[f]
    const color = colorInput[f]
    for (let v = 0; v < 4; v++) {
      const position = cornerPositions[faceIndices[v]]
      positions = positions.concat(position)
      colors = colors.concat(color)
    }
    let offset = 4 * f
    indices.push(offset + 0, offset + 1, offset + 2)
    indices.push(offset + 0, offset + 2, offset + 3)
  }

  indices = new Uint16Array(indices)
  positions = new Float32Array(positions)
  colors = new Float32Array(colors)

  return {
    positions,
    indices,
    colors,
  }
}
```

有了生成立方体顶点的方法，我们生成一个边长为 1 的正方体：

```js
const cube = createCube(1, 1, 1)
```

拿到了顶点的信息，就可以用我们熟悉的索引绘制方法来进行绘制了，这部分代码和之前一样，我们就不写了，看下效果：

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2018/9/30/16629a1a1667d97c~tplv-t2oaga2asx-watermark.awebp)

看到这个红色矩形，我们不禁有疑问：

- 我们定义的立方体的边长都是 1 ，也就是一个正方体，每个面应该是正方形，为什么渲染到屏幕后就成长方形了？

- 如何才能看到立方体的其他表面？

### 第一个问题的答案

这是因为，我们给 gl_Position 赋的坐标，在 渲染到屏幕之前，GPU 还会对其做一次坐标变换：视口变换。该变换会将 NDC 坐标转换成对应设备的视口坐标。

假设有一顶点 P（0.5，0.5，0.5，1）， gl_Position 接收到坐标后，会经历如下阶段：

首先执行透视除法，将顶点 P 的坐标从裁剪坐标系转换到 NDC 坐标系，转换后的坐标为： P1（0.5 / 1, 0.5 / 1, 0.5 / 1, 1 / 1）。由于 w 分量是 1， 所以 P1 和 P 的坐标一致。

接着，GPU 将顶点渲染到屏幕之前，对顶点坐标执行视口变换。假设我们的 canvas 视口宽度 300，高度 400，顶点坐标在 canvas 中心。那么 3D 坐标转换成 canvas 坐标的算法是:

canvas 坐标系 X 轴坐标 = NDC 坐标系下 X 轴坐标 * 300 / 2 = 0.5 * 150 = 75

canvas 坐标系 Y 轴坐标 = NDC 坐标系下 Y 轴坐标 * 400 / 2 = 0.5 * 200 = 100

所以会有一个问题，立方体的每个面宽度和高度虽然都是 1 ，但是渲染效果会随着显示设备的尺寸不同而不同。

这个问题该如何解决呢？这就引出了 WebGL 坐标系的一个重要变换：**投影变换**。

### 第二个问题的答案

因为我们绘制的是立方体，没有施加动画效果，所以我们只能看到立方体前表面，那如何看到其他表面呢？大家稍微一想就能知道，我们可以让立方体转动起来，转起来之后我们就能看到其他表面了。 那如何让立方体转动起来呢？ 这就引出了 WebGL 坐标系的另一个重要变换：模型变换。

针对这两个问题的解决方案是对顶点施加投影和模型变换，本节我们采用业界常用的变换算法，暂时不做算法原理的讲解，只讲如何使用，让我们的正方体可以正常渲染并且能转动起来。

> 请谨记：每个转换可以用一个矩阵来表示，转换矩阵相乘，得出的最终矩阵用来表示组合变换。大家先记住这点，在中级进阶中的数学矩阵及运算中我会详细讲解。

### 让立方体转动起来

- 引入**模型变换**让立方体可以转动，以便我们能观察其他表面。
- 引入**投影变换**让我们的正方体能够以正常比例渲染到目标设备，不再随视口的变化而拉伸失真。

为了引入这两个变换，我们需要引入**矩阵乘法**、**绕 X 轴旋转**、**绕 Y 轴旋转**、**正交投影**四个方法，如下：

```js
//返回一个单位矩阵
function identity() {}
//计算两个矩阵的乘积，返回新的矩阵。
function multiply(matrixLeft, matrixRight){}
//绕 X 轴旋转一定角度，返回新的矩阵。
function rotationX(angle) {}
//绕 Y 轴旋转一定角度，返回新的矩阵。
function rotateY(m, angle) {}
//正交投影，返回新的矩阵
function ortho({left, right, bottom, top, near, far}, target) {}
```

在顶点着色器中定义一个变换矩阵，用来接收 JavaScript 中传过来的模型投影变换矩阵，同时将变换矩阵左乘顶点坐标。

```js
const VERTEX_SHADER_SOURCE = `
  precision mediump float;
  attribute vec3 a_Position;
  attribute vec4 a_Color;
  varying vec4 v_Color;
  uniform mat4 u_Matrix;
  void main() {
    gl_Position = u_Matrix * vec4(a_Position, 1);
    v_Color = a_Color;
    gl_PointSize = 5.0;
  }
`
```

增加旋转动画效果：每隔 50 ms 分别绕 X 轴和 Y 轴转动 1 度，然后将旋转对应的矩阵传给顶点着色器。

```js
const dstMatrix = identity()
const tmpMatrix = identity()
let xAngle = 0
let yAngle = 0

const render = () => {
  xAngle += 1
  yAngle += 1
  // 先绕 y 轴旋转矩阵
  rotationY(deg2radians(yAngle), dstMatrix)
  // 再绕 x 轴旋转
  multiply(dstMatrix, rotationX(deg2radians(xAngle), tmpMatrix), dstMatrix)

  // 模型投影矩阵
  multiply(projectionMatrix, dstMatrix, dstMatrix)

  gl.uniformMatrix4fv(u_Matrix, false, dstMatrix)

  gl.clear(gl.COLOR_BUFFER_BIT)

  gl.drawElements(gl.TRIANGLES, indices.length, gl.UNSIGNED_SHORT, 0)

  requestAnimationFrame(render)
}
```

[![Edit 17-绘立方体](https://codesandbox.io/static/img/play-codesandbox.svg)](https://codesandbox.io/s/17-hui-li-fang-ti-9iv1z?fontsize=14&hidenavigation=1&theme=dark)

## 完整代码

```js
import { createProgram, createShader, getWebGLContext } from '@3dgl/utils'
import { identity, multiply, ortho, rotationX, rotationY } from '@3dgl/matrix'
import { deg2radians } from '@3dgl/math'


const canvas = document.querySelector<HTMLCanvasElement>('#canvas')

if (canvas === null) {
  throw new Error('canvas is null')
}

const gl = getWebGLContext(canvas)

/**
 * 顶点着色器
 */
const VERTEX_SHADER_SOURCE = `
  precision mediump float;
  attribute vec3 a_Position;
  attribute vec4 a_Color;
  varying vec4 v_Color;
  uniform mat4 u_Matrix;
  void main() {
    gl_Position = u_Matrix * vec4(a_Position, 1);
    v_Color = a_Color;
    gl_PointSize = 5.0;
  }
`
/**
 * 定义片元着色器
 */
const FRAG_SHADER_SOURCE = `
  precision mediump float;
  varying vec4 v_Color;
  void main() {
    gl_FragColor = v_Color;
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
const a_Color = gl.getAttribLocation(program, 'a_Color')
const u_Matrix = gl.getUniformLocation(program, 'u_Matrix')

/**
 * 顶点数据
 */
const positions = [
  /**
   * 前面
   */
  -0.5, -0.5, 0.5, 1, 0, 0, 1,
  0.5, -0.5, 0.5, 1, 0, 0, 1,
  0.5, 0.5, 0.5, 1, 0, 0, 1,
  -0.5, 0.5, 0.5, 1, 0, 0, 1,

  /**
   * 左面
   */
  -0.5, 0.5, 0.5, 0, 1, 0, 1,
  -0.5, 0.5, -0.5, 0, 1, 0, 1,
  -0.5, -0.5, -0.5, 0, 1, 0, 1,
  -0.5, -0.5, 0.5, 0, 1, 0, 1,

  /**
   * 右面
   */
  0.5, 0.5, 0.5, 0, 0, 1, 1,
  0.5, -0.5, 0.5, 0, 0, 1, 1,
  0.5, -0.5, -0.5, 0, 0, 1, 1,
  0.5, 0.5, -0.5, 0, 0, 1, 1,

  /**
   * 后面
   */
  0.5, 0.5, -0.5, 1, 0, 1, 1,
  0.5, -0.5, -0.5, 1, 0, 1, 1,
  -0.5, -0.5, -0.5, 1, 0, 1, 1,
  -0.5, 0.5, -0.5, 1, 0, 1, 1,

  /**
   * 上面
   */
  -0.5, 0.5, 0.5, 1, 1, 0, 1,
  0.5, 0.5, 0.5, 1, 1, 0, 1,
  0.5, 0.5, -0.5, 1, 1, 0, 1,
  -0.5, 0.5, -0.5, 1, 1, 0, 1,

  /**
   * 下面
   */
  -0.5, -0.5, 0.5, 0, 1, 1, 1,
  -0.5, -0.5, -0.5, 0, 1, 1, 1,
  0.5, -0.5, -0.5, 0, 1, 1, 1,
  0.5, -0.5, 0.5, 0, 1, 1, 1,
]

const indices = [
  0, 1, 2,
  0, 2, 3,

  4, 5, 6,
  4, 6, 7,

  8, 9, 10,
  8, 10, 11,

  12, 13, 14,
  12, 14, 15,

  16, 17, 18,
  16, 18, 19,

  20, 21, 22,
  20, 22, 23,
]

/**
 * 创建缓冲区
 */
const buffer = gl.createBuffer()
/**
 * 绑定缓冲区为当前缓冲
 */
gl.bindBuffer(gl.ARRAY_BUFFER, buffer)
/**
 * 设置 a_Position 属性从缓冲区读取数据方式
 */
gl.vertexAttribPointer(a_Position, 3, gl.FLOAT, false, 4 * 7, 0)
/**
 * 设置 a_Color 属性从缓冲区读取数据方式
 */
gl.vertexAttribPointer(a_Color, 4, gl.FLOAT, false, 4 * 7, 3 * 4)
/**
 * 向缓冲区传递数据
 */
gl.bufferData(gl.ARRAY_BUFFER, new Float32Array(positions), gl.STATIC_DRAW)

gl.enableVertexAttribArray(a_Position)
gl.enableVertexAttribArray(a_Color)

/**
 * 创建索引
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
 * 清空屏幕
 */
gl.clearColor(0.0, 0.0, 0.0, 0.1)
gl.clear(gl.COLOR_BUFFER_BIT)

/**
 * 隐藏背面
 */
gl.enable(gl.CULL_FACE)

const aspect = canvas.width / canvas.height
/**
 * 计算正交投影矩阵
 */
const projectionMatrix = ortho({
  left: -aspect * 2,
  right: aspect * 2,
  bottom: -2,
  top: 2,
  near: 100,
  far: -100,
})

const dstMatrix = identity()
const tmpMatrix = identity()
let xAngle = 0
let yAngle = 0

const render = () => {
  xAngle += 1
  yAngle += 1
  // 先绕 y 轴旋转矩阵
  rotationY(deg2radians(yAngle), dstMatrix)
  // 再绕 x 轴旋转
  multiply(dstMatrix, rotationX(deg2radians(xAngle), tmpMatrix), dstMatrix)

  // 模型投影矩阵
  multiply(projectionMatrix, dstMatrix, dstMatrix)

  gl.uniformMatrix4fv(u_Matrix, false, dstMatrix)

  gl.clear(gl.COLOR_BUFFER_BIT)

  gl.drawElements(gl.TRIANGLES, indices.length, gl.UNSIGNED_SHORT, 0)

  requestAnimationFrame(render)
}

render()
```