# 绘制正方体

## WebGL 坐标系

3D 形体的顶点坐标系需要包含深度信息 z 轴坐标。所以我们先了解一下 WebGL 坐标系的概念。

WebGL 采用了左手坐标系，x 轴向右为正，y 轴向上为正，z 轴沿着屏幕往里为正。

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2018/9/29/16624dfadfcfef91~tplv-t2oaga2asx-watermark.awebp)

WebGL 是遵循右手坐标系，但仅仅是遵循，是期望大家遵守的规范。其实 WebGL 内部（裁剪坐标系）是基于左手坐标系的，z 轴沿屏幕向里为正方向。