# 绘制一个点

## 编写着色器程序

```js
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
```

在 JS 中编写着色器程序，我们通常写在字符串里，或者 script 标签中通过 innerHTML 拿到着色器字符串。

解释：

- gl_Position：顶点的**裁剪坐标系**坐标，包含 X, Y, Z，W 四个坐标分量，顶点着色器接收到这个坐标之后，对它进行透视除法，即将各个分量同时除以 W，转换成 **NDC 坐标**，NDC 坐标每个分量的取值范围都在 `[-1, 1]` 之间，GPU 获取这个属性值作为顶点的最终位置进行绘制。

- gl_PointSize：绘制到屏幕的点的大小，需要注意的是，gl_PointSize 只有在绘制图元是点的时候才会生效。当我们绘制线段或者三角形的时候，gl_PointSize是不起作用的。

- gl_FragColor：片元（像素）颜色，包含 R, G, B, A 四个颜色分量，且每个分量的取值范围在 `[0,1]` 之间，GPU 获取这个值作为像素的最终颜色进行着色。

- vec4：包含四个浮点元素的**容器类型**，vec 是 vector（向量）的单词简写，vec4 代表包含 4 个浮点数的向量。此外，还有 vec2、vec3 等类型，代表包含2个或者3个浮点数的容器。

## 创建着色器对象

着色器分为两种：顶点（vertex）着色器和片元（fragment）着色器

初始化着色器对象分为三个步骤：

1. 创建着色器对象
2. 将着色器源码分配给着色器对象
3. 编译着色器 

创建着色器对象 `gl.createShader()` 可以通过传入 `gl.VERTEX_SHADER` 和 `gl.FRAGMENT_SHADER` 来分别创建顶点着色器和片元着色器。

我们要分别对顶点着色器和片元着色器执行以上三个步骤。


```js
/**
 * 创建着色器
 */
const vertexShader = gl.createShader(gl.VERTEX_SHADER)
const fragShader = gl.createShader(gl.FRAGMENT_SHADER)

/**
 * 将着色器源码分配给着色器对象
 */
gl.shaderSource(vertexShader, VERTEX_SHADER_SOURCE)
gl.shaderSource(fragShader, FRAG_SHADER_SOURCE)

/**
 * 编译着色器
 */
gl.compileShader(vertexShader)
gl.compileShader(fragShader)
```

## 创建着色器程序

着色器对象创建完毕，接下来我们开始创建着色器程序

创建着色器程序主要分为以下三步：
1. 创建着色器程序
2. 将着色器对象挂载到着色器程序上
3. 链接着色器程序

```js
/**
 * 创建着色器程序
 */
const program = gl.createProgram()
/**
 * 将着色器对象挂载到着色器程序上
 */
gl.attachShader(program, vertexShader)
gl.attachShader(program, fragShader)

/**
 * 链接着色器程序
 */
gl.linkProgram(program)
```

有时候一个 WebGL 应用包含多个 program，所以在使用某个 program 绘制之前，我们要先启用它。

```js
/**
 * 使用刚创建好的着色器程序
 */
gl.useProgram(program)
```

## 清空绘制区

```js
/**
 * 清空颜色缓冲区
 */
gl.clearColor(0.0, 0.0, 0.0, 0.1)
/**
 * 用颜色缓冲区设置背景色
 */
gl.clear(gl.COLOR_BUFFER_BIT)
```

## 开始绘制

```js
/**
 * 绘制
 */
gl.drawArrays(gl.POINTS, 0, 1)
```
我们通常会使用 `gl.drawArrays(mode, first, count)` 函数来进行绘制。

```js
gl.drawArrays(mode, first, count);

参数：
- mode，代表图元类型，这里我们绘制一个点，所以是 gl.POINTS。
- first，代表从第几个点开始绘制。
- count，代表绘制的点的数量。
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
const gl = canvas.getContext('webgl')

if (gl === null) {
  throw new Error('fail to get rendering context of WebGL')
}

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
 * 创建着色器
 */
const vertexShader = gl.createShader(gl.VERTEX_SHADER)
const fragShader = gl.createShader(gl.FRAGMENT_SHADER)

/**
 * 将着色器源码分配给着色器对象
 */
gl.shaderSource(vertexShader, VERTEX_SHADER_SOURCE)
gl.shaderSource(fragShader, FRAG_SHADER_SOURCE)

/**
 * 编译着色器
 */
gl.compileShader(vertexShader)
gl.compileShader(fragShader)

/**
 * 创建着色器程序
 */
const program = gl.createProgram()
/**
 * 将着色器对象挂载到着色器程序上
 */
gl.attachShader(program, vertexShader)
gl.attachShader(program, fragShader)

/**
 * 链接着色器程序
 */
gl.linkProgram(program)

/**
 * 使用刚创建好的着色器程序
 */
gl.useProgram(program)

/**
 * 清空颜色缓冲区
 */
gl.clearColor(0.0, 0.0, 0.0, 0.1)
/**
 * 用颜色缓冲区设置背景色
 */
gl.clear(gl.COLOR_BUFFER_BIT)

/**
 * 绘制
 */
gl.drawArrays(gl.POINTS, 0, 1)
```

## 总结

### GLSL

- gl_Position： 内置变量，用来设置顶点坐标。
- gl_PointSize： 内置变量，用来设置顶点大小。
- gl_FragColor： 内置变量，用来设置像素颜色。
- vec4：4 维向量容器，可以存储 4 个浮点数。

### WebGL API 着色器相关

- createShader：创建着色器对象
- shaderSource：提供着色器源码
- compileShader：编译着色器对象
- createProgram：创建着色器程序
- attachShader：绑定着色器对象
- linkProgram：链接着色器程序
- useProgram：启用着色器程序

### WebGL API 绘制相关

- drawArrays: 用指定的图元进行绘制。

### WebGL 图元

- gl.POINTS: 将绘制图元类型设置成点图元。
