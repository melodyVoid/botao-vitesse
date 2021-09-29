# 清空绘图区

主要流程如下：

1. 获取 canvas 元素
2. 获取 webgl 绘制上下文
3. 设置背景色
4. 清空 canvas

WebGL 是基于 OpenGL ES 的，`gl.clearColor()` 函数对应着 OpenGL ES 2.0 或 OpenGL 中的 `glClearColor()` 函数。

```js
gl.clearColor(0.0, 0.0, 0.0, 0.1)
``` 
接收的参数分别是 r、g、b、a。取值都是 0.0 ~ 1.0 的闭区间 （[0.0, 1.0]）如果小于 0.0 或者大于 1.0 则分别截断为 0.0 或 1.0。

WebGL 中的颜色值可以通过 `普通颜色分量值 / 255` 得到。

`gl.clearColor()` 传入的颜色值实际上就是在设置背景色，一旦指定了背景色后，背景色就会进驻存在 WebGL 系统中，在下一次调用 `gl.clearColor()` 前都不会变。

`gl.clear(gl.COLOR_BUFFER_BIT)` 是用之前设置的背景色来清空**颜色缓冲区（color buffer）**。`gl.COLOR_BUFFER_BIT` 就是在告诉 WebGL 清空颜色缓冲区。

如果没有指定背景色，默认为无色 `gl.clearColor(0.0, 0.0, 0.0, 0.0)`

[![Edit 01-清空绘图区](https://codesandbox.io/static/img/play-codesandbox.svg)](https://codesandbox.io/s/01-qing-kong-hui-tu-qu-ln1li?fontsize=14&hidenavigation=1&theme=dark)

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
 * 清空颜色缓冲区
 */
gl.clearColor(0.0, 0.0, 0.0, 0.1)
/**
 * 用颜色缓冲区设置背景色
 */
gl.clear(gl.COLOR_BUFFER_BIT)

```