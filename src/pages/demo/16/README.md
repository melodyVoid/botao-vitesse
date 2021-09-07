# 纹理贴图

## 为什么需要纹理贴图

为图形增加色彩仅仅是用了简单的单色和渐变色，但是实际应用中往往需要一些丰富多彩的图案，我们不可能用代码来生成这些图案，费时费力，效果也不好。通常我们会借助一些图形软硬件（比如照相机、手机、PS等）准备好图片素材，然后在 WebGL 中把图片应用到图形表面。

## 纹理图片格式

WebGL 对图片素材是有严格要求的，图片的宽度和高度必须是 2 的 N 次幂，比如 16 x 16，32 x 32，64 x 64 等。实际上，不是这个尺寸的图片也能进行贴图，但是这样会使得贴图过程更复杂，从而影响性能，所以我们在提供图片素材的时候最好参照这个规范。

## 纹理坐标系

纹理也有一套自己的坐标系统，为了和顶点坐标加以区分，通常把纹理坐标称为 `UV`，`U` 代表横轴坐标，`V` 代表纵轴坐标。

### 图片坐标系统的特点

- **左上角**为原点（0, 0）。
- 向右为横轴正方向，横轴最大值为 1，即横轴坐标范围 [0, 1]。
- 向下为纵轴正方向，纵轴最大值为 1，即纵轴坐标范围 [0, 1]。

### 纹理坐标系统不同于图片坐标系统，它的特点是：

- **左下角**为原点（0, 0）。
- 向右为横轴正方向，横轴最大值为 1，即横轴坐标范围 [0, 1]。
- 向上为纵轴正方向，纵轴最大值为 1，即纵轴坐标范围 [0, 1]。

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/1/25/1688415c89af299b~tplv-t2oaga2asx-watermark.awebp)

纹理坐标系统可以理解为一个边长为 1 的正方形。

## 贴图

### 准备图片

按照规范所讲，我们首先准备一张符合要求的图片，这里自己制作一个尺寸为宽高分别是 2 的 7 次方，即 128 x 128 的图片。

### 着色器

本例中的片元着色器，不再是接收单纯的颜色了，而是接收纹理图片对应坐标的颜色值，所以我们的着色器要能够做到如下几点：

- 顶点着色器接收顶点的 `UV` 坐标，并将 `UV` 坐标传递给片元着色器。
- 片元着色器要能够接收顶点插值后的 `UV` 坐标，同时能够在纹理资源找到对应坐标的颜色值。

#### 顶点着色器

1. 增加一个名为 `v_Uv` 的 attribute 变量，接收 JavaScript 传递过来的 UV 坐标。
2. 增加一个 varying 变量 `v_Uv`，将 UV 坐标插值化，并传递给片元着色器。

```js
const VERTEX_SHADER_SOURCE = `
	precision mediump float;
	attribute vec2 a_Position;
	attribute vec2 a_Screen_Size;
	attribute vec2 a_Uv;
	varying vec2 v_Uv;
	void main() {
		vec2 position = (a_Position / a_Screen_Size) * 2.0 - 1.0;
		position = position * vec2(1, 0, -1.0);
		gl_Position = vec4(position, 0, 1);
		v_Uv = a_Uv;
	}
`
```

#### 片元着色器

1. 增加一个 varying 变量 `v_Uv`，接收顶点着色器插值过来的 UV 坐标。
2. 增加一个 sampler2D 类型的全局变量 `texture`，用来接收 JavaScript 传递过来的纹理资源（图片数据）。

```js
const FRAG_SHADER_SOURCE = `
	precision mediump float;
	varying vec2 v_Uv;
	uniform sampler2D texture;
	void main() {
		gl_FragColor = texture2D(texture, vec2(v_Uv.x, v_Uv.y))
	}
`
```

- `varying vec2 v_Uv`  接收顶点着色器传递过来的 uv 值。 
- `uniform sampler2D texture` 接收 JavaScript 传递过来的纹理。
- `gl_FragColor = texture2D(texture, vec2(v_Uv.x, v_Uv.y))` 提取纹理对应的 uv 坐标上的颜色，赋值给当前片元（像素）。

#### JavaScript部分

我们首先要将纹理图片加载到内存中：

```js
const img = new Image()
img.onload = textureLoadedCallback
img.src = ''
```

图片加载完成之后才能执行纹理的操作，我们将纹理操作放在图片加载完成后的回调函数中，即 `textureLoadedCallback`。

需要注意的是，我们使用 canvas 读取图片数据是受浏览器跨域限制的，所以首先要解决跨域问题。

针对图片跨域问题我们可以采用三种方式来解决：

- **设置允许 Chrome 跨域加载资源**

  在本地开发阶段，我们可以设置 Chrome 浏览器允许加载跨域资源，这样就可以使用磁盘地址来访问页面了。
	mac 设置如下：
	```
	open -n /Applications/Google\ Chrome.app/ --args --disable-web-security --user-data-dir(指定目录,例如 = /user/Documents)
	```

- **图片资源和页面资源放在同一个域名下**

	除了设置 Chrome，我们还可以将图片资源和页面资源部署在同一域名下，这样就不存在跨域问题了。

- **为图片资源设置跨域响应头**

  实际生产环境中，图片资源往往部署在 CDN 上，图片和页面分属不同域，这种情况的跨域访问我们就需要正面解决了。

	假设我们的图片资源所属域名为：https://cdn-pic.com，页面所属域名为 https://test.com。

	解决方法如下：

	首先：为图片资源设置跨域响应头：

	```
	Access-Control-Allow-Origin：`https://test.com`
	```

	其次：在图片加载时，为 img 设置 crossOrigin 属性。

	```js
	const img = new Image()
	img.crossOrigin = ''
	img.src = 'https://cdn-pic.com/test.jpg'
	```

	做完这两步，我们就可以真正的加载跨域图片了。 解决了图片加载跨域问题，我们就可以开始纹理贴图了。

我们定义六个顶点，这六个顶点能够组成一个矩形，并为顶点指定纹理坐标。

```js
const positions = [
	30, 30, 0, 0, // v0
	30, 370, 0, 1, // v1
	370, 370, 1, 1, // v2
	30, 30, 0, 0, // v0
	370, 370, 1, 1, // v2
	370, 30, 1, 0, // v3 
]
```

为着色器传递数据。

加载图片

```js
const img  = new Image()
img.onload = textureLoadedCallback
img.src = ''
```

图片加载完成后，我们进行如下操作：

首先：激活 0 号纹理通道 `gl.TEXTURE0`，0 号纹理通道是默认值，本例也可以不设置。

```js
gl.activeTexture(gl.TEXTURE0)
```

然后创建一个纹理对象：

```js
const texture = gl.createTexture()
```

之后将创建好的纹理对象 texture 绑定到当前纹理绑定点上，即 `gl.TEXTURE_2D`。绑定完之后对当前纹理对象的所有操作，都将基于 texture 对象，直到重新绑定。

```js
gl.bindTexture(gl.TEXTURE_2D, texture)
```

为片元着色器传递图片数据：

```js
gl.texImage2D(gl.TEXTURE_2D, 0, gl.RGBA, gl.UNSIGNED_BYTE, img)
```

`gl.texImage2D` 方法是一个重载方法，其中有一些参数可以省略。

| 参数 | 含义 |
| --- | --- |
| target | 纹理类型，TEXTURE_2D 代表2维纹理 |
| level | 表示多级分辨率的纹理图像的级数，若只有一种分辨率，则 level 设为 0，通常我们使用一种分辨率 |
| components | 纹理通道数，通常我们使用 RGBA 和 RGB 两种通道 |
| width | 纹理宽度，可省略 |
| height | 纹理高度，可省略 |
| border | 边框，通常设置为0，可省略 |
| format | 纹理映射的格式 |
| type | 纹理映射的数据类型 |
| pixels | 纹理图像的数据 |

上面这段代码的意思是，我们将 img 变量指向的图片数据传递给片元着色器，取对应纹理坐标的 RGBA 四个通道值，赋给片元，每个通道的数据格式是无符号单字节整数。

接下来，我们设置图片在放大或者缩小时采用的算法 `gl.LINEAR`。

> gl.LINEAR 代表采用最靠近象素中心的四个象素的加权平均值，这种效果表现的更加平滑自然。 gl.NEAREST 采用最靠近象素中心的纹素，该算法可能使图像走样，但是执行效率高，不需要额外的计算。

```js
gl.texParameterf(gl.TEXTURE_2D, gl.TEXTURE_MAG_FILTER, gl.LINEAR)
gl.texParameterf(gl.TEXTURE_2D, gl.TEXTURE_MIN_FILTER, gl.LINEAR)
```

之后为片元着色器传递 0 号纹理单元：

```js
gl.uniform1i(uniformTexture, 0)
```

> 这里我们为片元着色器的 texture 属性传递 0，此处应该与激活纹理时的通道值保持一致。

图片作为纹理的渲染效果如下：

![](https://files.catbox.moe/5ejb65.png)

可以看到，我们绘制的矩形表面贴上了纹理。

思考：为什么只是指定了三角形的顶点对应的 UV 坐标，GPU 就能够将纹理图片的其他坐标的颜色贴到三角形表面呢？

渲染管线光栅化环节上，GPU 处理两件事情：

- 计算图元覆盖了哪些像素。
- 根据顶点着色器的定点位置计算每个像素的纹理坐标的插值。

> 注：片元可以理解为像素。

光栅化结束后，来到片元着色器，片元着色器此时知道每个像素对应的 UV 坐标，根据当前像素的 UV 坐标，找到纹理资源对应坐标的颜色信息，赋值给当前像素，从而能够为图元表面的每个像素贴上正确的纹理颜色。

## 注意事项

- 图片最好满足 2^m x 2^n 的尺寸要求。
- 图片数据首先加载到内存中，才能够在纹理中使用。
- 图片资源加载前要先解决跨域问题。


[![Edit 16-纹理贴图](https://codesandbox.io/static/img/play-codesandbox.svg)](https://codesandbox.io/s/16-wen-li-tie-tu-4vvng?fontsize=14&hidenavigation=1&theme=dark)

## 完整代码

```ts
import {
  getWebGLContext,
  createShader,
  createProgram,
  loadTexture,
} from '@3dgl/utils'
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
  attribute vec2 a_Position;
  attribute vec2 a_Screen_Size;
  attribute vec2 a_Uv;
  varying vec2 v_Uv;
  void main() {
    vec2 position = (a_Position / a_Screen_Size) * 2.0 - 1.0;
    position = position * vec2(1.0, -1.0);
    gl_Position = vec4(position, 0, 1);
    v_Uv = a_Uv;
  }
`

/**
 * 定义片元着色器
 */
const FRAG_SHADER_SOURCE = `
  precision mediump float;
  varying vec2 v_Uv;
  uniform sampler2D u_Texture;
  void main() {
    gl_FragColor = texture2D(u_Texture, vec2(v_Uv.x, v_Uv.y));
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
const a_Uv = gl.getAttribLocation(program, 'a_Uv')
/**
 * 找到着色器中的全局变量 u_Texture
 */
const u_Texture = gl.getUniformLocation(program, 'u_Texture')

/**
 * 顶点数据
 */
const positions = [
  30,
  30,
  0,
  0, // v0
  30,
  370,
  0,
  1, // v1
  370,
  370,
  1,
  1, // v2
  30,
  30,
  0,
  0, // v0
  370,
  370,
  1,
  1, // v2
  370,
  30,
  1,
  0, // v3
]

gl.vertexAttrib2f(a_Screen_Size, canvas.width, canvas.height)

gl.enableVertexAttribArray(a_Position)
gl.enableVertexAttribArray(a_Uv)

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
gl.vertexAttribPointer(a_Position, 2, gl.FLOAT, false, 4 * (2 + 2), 0)
/**
 * 设置 a_Uv 属性从缓冲区读取数据方式
 */
gl.vertexAttribPointer(a_Uv, 2, gl.FLOAT, false, 4 * (2 + 2), 4 * 2)

/**
 * 向缓冲区传入数据
 */
gl.bufferData(gl.ARRAY_BUFFER, new Float32Array(positions), gl.STATIC_DRAW)

/**
 * 清空绘图区
 */
gl.clearColor(0.0, 0.0, 0.0, 0.1)

const render = () => {
  gl.clear(gl.COLOR_BUFFER_BIT)
  /**
   * 绘制
   */
  gl.drawArrays(gl.TRIANGLES, 0, positions.length / 4)
}

render()

loadTexture(gl, 'https://files.catbox.moe/jiekom.jpeg', u_Texture, () => {
  render()
})
```