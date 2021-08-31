# 绘制线段

## 线段图元的分类

线段图元分为三种：

1. LINES：基本线段。
2. LINE_STRIP：带状线段。
3. LINE_LOOP：环状线段。

## LINES 图元

LINES 图元称为基本线段图元，绘制每一条线段都需要明确指定构成线段的两个端点。

我们还是通过每次点击产生一个点，并将点击位置坐标放进 positions 数组中。

> 注意，我们的坐标还是相对于屏幕坐标系，顶点着色器中会将屏幕坐标系转换到裁剪坐标系，也就是坐标区间在[-1, 1]之间。

## 点击逻辑

```js
canvas.addEventListener('click', e => {
  const x = e.offsetX
  const y = e.offsetY
  // 两个坐标为一组
  positions.push(x, y)

  if (positions.length > 0) {
    /**
     * 向当前缓冲区写入数据
     */
    gl.bufferData(gl.ARRAY_BUFFER, new Float32Array(positions), gl.STATIC_DRAW)
    /**
     * 设置颜色
     */
    gl.uniform4f(u_Color, 255, 0, 0, 1)
    gl.clear(gl.COLOR_BUFFER_BIT)
    /**
     * 绘制线段，这里的图元我们设置为 gl.LINES
     */
    gl.drawArrays(gl.LINES, 0, positions.length / 2)
  }
})
```

每点击两次就会绘制一条线段，但是线段是互相独立的。

如果我们想绘制线段带（gl.LINE_STRIP）或者线段环（gl.LINE_LOOP）可以设置对应的图元。

## 完整代码

```js
import { getWebGLContext, createShader, createProgram } from '@3dgl/utils'
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
    void main() {
      vec2 position = (a_Position / a_Screen_Size) * 2.0 - 1.0;
      position = position * vec2(1.0, -1.0);
      gl_Position = vec4(position, 0.0, 1.0);
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
 * 初始化程序
 */
const { program } = createProgram(gl, vertexShader, fragShader)

/**
 * 使用 program
 */
gl.useProgram(program)

const a_Position = gl.getAttribLocation(program, 'a_Position')
const a_Screen_Size = gl.getAttribLocation(program, 'a_Screen_Size')
const u_Color = gl.getUniformLocation(program, 'u_Color')

/**
 * 为顶点着色器中的 a_Screen_Size 传递 canvas 的宽高信息
 */
gl.vertexAttrib2f(a_Screen_Size, canvas.width, canvas.height)

/**
 * 创建缓冲区
 */
const buffer = gl.createBuffer()

/**
 * 绑定缓冲区
 */
gl.bindBuffer(gl.ARRAY_BUFFER, buffer)

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
 * 清空绘图区
 */
gl.clearColor(0.0, 0.0, 0.0, 0.1)
gl.clear(gl.COLOR_BUFFER_BIT)

/**
 * 点击 canvas 动态绘制线段
 */
canvas.value.addEventListener('click', e => {
  const x = e.offsetX
  const y = e.offsetY
  // 两个坐标为一组
  positions.push(x, y)

  if (positions.value.length > 0) {
    /**
     * 向当前缓冲区写入数据
     */
    gl.bufferData(gl.ARRAY_BUFFER, new Float32Array(positions), gl.STATIC_DRAW)
    /**
     * 设置颜色
     */
    gl.uniform4f(u_Color, 255, 0, 0, 1)
    gl.clear(gl.COLOR_BUFFER_BIT)
    /**
     * 绘制线段，试着将 gl.LINES 改成 gl.LINE_STRIP 和 gl.LINE_LOOP
     */
    gl.drawArrays(gl.LINES, 0, positions.length / 2)
  }
})
```
