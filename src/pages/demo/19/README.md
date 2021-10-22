# 绘制环形

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2018/12/27/167ed9b3702ca350~tplv-t2oaga2asx-watermark.awebp)

## 绘制方法

建立两个圆，一个内圆，一个外圆，划分n个近似于扇形的三角形，每个三角形的两条边都会和内圆和外圆相交，产生四个交点，这四个交点组成一个近似矩形，然后将近似矩形划分成两个三角形：

```javascript
function createRingVertex(x, y, innerRadius, outerRadius, n) {
    var positions = [];
    var color = randomColor();
    for (var i = 0; i <= n; i++) {
        if (i % 2 == 0) {
            color = randomColor();
        }
        var angle = i * Math.PI * 2 / n;
        positions.push(x + innerRadius * sin(angle), y + innerRadius * cos(angle), color.r, color.g, color.b, color.a);
        positions.push(x + outerRadius * sin(angle), y + outerRadius * cos(angle), color.r, color.g, color.b, color.a);
    }
    var indices = [];
    for (var i = 0; i < n; i++) {
        var p0 = i * 2;
        var p1 = i * 2 + 1;
        var p2 = (i + 1) * 2 + 1;
        var p3 = (i + 1) * 2;
        if (i == n - 1) {
          p2 = 1;
          p3 = 0;
        }
        indices.push(p0, p1, p2, p2, p3, p0);
    }
    return { 
        positions: positions, 
        indices: indices 
    };
}
```

上面这个方法能够根据内圆半径和外圆半径以及三角形的数量返回顶点数组和索引数组，我们生成 100 个三角形的信息。

```javascript
var geo = createRingVertex(100, 100, 20, 50, 100);
```

为了节省空间，我们采用索引绘制：

```javascript
gl.drawElements(gl[currentType], indices.length, gl.UNSIGNED_SHORT, 0);
```

## 完整代码
```javascript


```