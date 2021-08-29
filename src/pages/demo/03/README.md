# 绘制一个点优化

在前面[绘制一个点](/demo/02)的示例中，我们在写顶点着色器和片元着色器的代码时，写了很多相同的逻辑，现在我们把它们封装一下。

## 获取 WebGL 绘制上下文

在某些浏览器中，我们还需要做下兼容处理，加上实验前缀，然后我们将这些细节隐藏起来，外部直接调用 `getWebGLContext` 函数就能拿到 WebGL 的绘制上下文了。

而且这些工具函数都是有类型的，能提供更好的智能感知（intellisense）。

```ts
/**
 * 获取 WebGL 绘制上下文对象
 */
export const getWebGLContext = (element: HTMLElement | null) => {
  if (element === null) {
    throw new Error('element is null')
  }
  const gl =
    (element as HTMLCanvasElement).getContext('webgl') ||
    (element as HTMLCanvasElement).getContext('experimental-webgl')

  if (gl === null) {
    throw new Error('fail to get rendering context of WebGL')
  }
  return gl
}
```

## 封装初始化着色器函数

回忆一下初始化着色器对象的步骤：

1. 创建着色器对象
2. 将着色器源码分配给着色器对象
3. 编译着色器

创建顶点着色器和片元着色器的唯一不同是：传入 `gl.createShader()` 的类型不同。

```ts
/**
 * 创建并初始化着色器
 */
export const createShader = (
  gl: WebGLRenderingContext,
  type: number,
  source: string
): WebGLShader => {
  // 创建着色器对象
  const shader = gl.createShader(type)
  if (shader === null) {
    throw new Error('fail to createShader')
  }
  // 将着色器源码分配给着色器对象
  gl.shaderSource(shader, source)

  // 编译着色器
  gl.compileShader(shader)

  // 检测编译是否正常
  const success = gl.getShaderParameter(shader, gl.COMPILE_STATUS)
  if (success) {
    return shader
  }
  console.error(gl.getShaderInfoLog(shader))
  gl.deleteShader(shader)
  throw new Error('fail to compile shader')
}
```

## 封装创建着色器程序工具函数

回忆一下创建着色器程序的步骤：

1. 创建着色器程序
2. 将着色器对象挂载到着色器程序上
3. 链接着色器程

```ts
/**
 * 创建 Program
 */
export const createProgram = (
  gl: WebGLRenderingContext,
  vertexShader: WebGLShader,
  fragShader: WebGLShader
): WebGLProgram => {
  // 创建程序
  const program = gl.createProgram()

  if (program === null) {
    throw new Error('fail to create program')
  }

  // 绑定着色器
  gl.attachShader(program, vertexShader)
  gl.attachShader(program, fragShader)

  // 链接程序
  gl.linkProgram(program)

  // 是否链接成功
  const success = gl.getProgramParameter(program, gl.LINK_STATUS)

  if (success) {
    return {
      program,
    }
  }
  // 链接失败
  const errorLog = gl.getProgramInfoLog(program)
  gl.deleteProgram(program)
  throw errorLog
}
```

有个这三个工具函数，我们再写绘制一个点的代码就会简单很多。工具函数已经发布到 npm 上，👉 [@3dgl/utils](https://www.npmjs.com/package/@3dgl/utils)

## 优化后的代码

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
	void main() {
		gl_Position = vec4(0.0, 0.0, 0.0, 1.0);
		gl_PointSize = 10.0;
	}
`
/**
 * 定义片元着色器
 */
const FRAG_SHADER_SOURCE = `
	void main() {
		gl_FragColor = vec4(0.0, 0.0, 1.0, 1.0);
	}
`

/**
 * 初始化着色器
 */
const vertexShader = createShader(gl, gl.VERTEX_SHADER, VERTEX_SHADER_SOURCE)
const fragShader = createShader(gl, gl.FRAGMENT_SHADER, FRAG_SHADER_SOURCE)

/**
 * 创建着色器程序
 */
const { program } = createProgram(gl, vertexShader, fragShader)

/**
 * 使用着色器程序
 */
gl.useProgram(program)

/**
 * 清空绘图区
 */
gl.clearColor(0.0, 0.0, 0.0, 0.1)
gl.clear(gl.COLOR_BUFFER_BIT)

/**
 * 绘制
 */
gl.drawArrays(gl.POINTS, 0, 1)
```
