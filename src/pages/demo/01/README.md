```ts
/**
 * 获取 canvas 元素
 */
const canvas = document.getElementById('canvas')
/**
 * 获取 webgl 绘制上下文
 */
const gl = canvas.value.getContext('webgl')

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