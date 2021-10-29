<template>
  <div>
    <h2> 魔方</h2>
    <div id="three">
    </div>
  </div>
</template>

<script>
import * as THREE from 'three'

import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls'
import { nextTick } from 'vue'

window.requestAnimFrame = (function() { // 如果有变化则可能还需要requestAnimationFrame刷新
  return window.requestAnimationFrame
      || window.mozRequestAnimationFrame
      || window.webkitRequestAnimationFrame
      || window.msRequestAnimationFrame
      || window.webkitRequestAnimationFrame
})()

// 创建展示场景所需的各种元素
const cubeParams = { // 魔方参数
  x: 0,
  y: 0,
  z: 0,
  num: 3,
  len: 50,
  colors: ['rgba(255,193,37,1)', 'rgba(0,191,255,1)',
    'rgba(50,205,50,1)', 'rgba(178,34,34,1)',
    'rgba(255,255,0,1)', 'rgba(255,255,255,1)'],
}

const xLine = new THREE.Vector3(1, 0, 0)// X轴正方向
const xLineAd = new THREE.Vector3(-1, 0, 0)// X轴负方向
const yLine = new THREE.Vector3(0, 1, 0)// Y轴正方向
const yLineAd = new THREE.Vector3(0, -1, 0)// Y轴负方向
const zLine = new THREE.Vector3(0, 0, 1)// Z轴正方向
const zLineAd = new THREE.Vector3(0, 0, -1)// Z轴负方向

let scene = null
let cubes = null
export default {
  data() {
    return {
      renderer: null,
      width: null,
      height: null,
      controls: null,
      camera: null,
      // 魔方转动
      isRotating: false,
      // 碰撞检测
      intersect: null, // 碰撞光线穿过的元素
      normalize: null, // 触发平面法向量
      // 开始点
      startPoint: null,
      movePoint: null,
      mouse: new THREE.Vector2(), // 存储鼠标坐标或者触摸坐标
      initStatus: [], // 魔方初始状态
      rayCaster: new THREE.Raycaster(),
      controller: null,
    }
  },
  mounted() {
    nextTick(() => {
      this.threeStart()
    })
  },
  methods: {

    // 开始
    threeStart() {
      this.initScene()
      this.initLight()
      this.initObject()
      this.initCamera()
      this.initThree()
      this.render()

      // 监听鼠标事件
      this.renderer.domElement.addEventListener('mousedown', this.startCube, false)
      this.renderer.domElement.addEventListener('mousemove', this.moveCube, false)
      this.renderer.domElement.addEventListener('mouseup', this.stopCube, false)
      // 监听触摸事件
      this.renderer.domElement.addEventListener('touchstart', this.startCube, false)
      this.renderer.domElement.addEventListener('touchmove', this.moveCube, false)
      this.renderer.domElement.addEventListener('touchend', this.stopCube, false)

      this.controls = new OrbitControls(this.camera, this.renderer.domElement)// 创建控件对象
      this.controls.addEventListener('change', this.render)// 监听鼠标、键盘事件
    },

    // 获取操作焦点以及该焦点所在平面的法向量
    getIntersects(event) {
      // 触摸事件和鼠标事件获得坐标的方式有点区别
      // 将鼠标位置归一化为设备坐标。x 和 y 方向的取值范围是 (-1 to +1)
      // http://www.webgl3d.cn/threejs/docs/#api/zh/core/Raycaster
      if (event.touches) {
        const touch = event.touches[0]
        this.mouse.x = (touch.clientX / this.width) * 2 - 1
        this.mouse.y = -(touch.clientY / this.height) * 2 + 1
      }
      else {
        this.mouse.x = (event.clientX / this.width) * 2 - 1
        this.mouse.y = -(event.clientY / this.height) * 2 + 1
      }
      console.log(this.mouse)
      this.rayCaster.setFromCamera(this.mouse, this.camera)
      // Raycaster方式定位选取元素，可能会选取多个，以第一个为准
      const intersects = this.rayCaster.intersectObjects(scene.children)
      if (intersects.length) {
        try {
          console.log(intersects)
          if (intersects[0].object.cubeType === 'coverCube') {
            this.intersect = intersects[1]
            this.normalize = intersects[0].face.normal
          }
          else {
            this.intersect = intersects[0]
            this.normalize = intersects[1].face.normal
          }
        }
        catch (err) {
          // nothing
        }
      }
    },

    // 获取数组中的最小值
    min(arr) {
      let min = arr[0]
      for (let i = 1; i < arr.length; i++) {
        if (arr[i] < min)
          min = arr[i]
      }
      return min
    },

    // 获得旋转方向
    getDirection(vector3) {
      let direction
      // 判断差向量和x、y、z轴的夹角
      const xAngle = vector3.angleTo(xLine)
      const xAngleAd = vector3.angleTo(xLineAd)
      const yAngle = vector3.angleTo(yLine)
      const yAngleAd = vector3.angleTo(yLineAd)
      const zAngle = vector3.angleTo(zLine)
      const zAngleAd = vector3.angleTo(zLineAd)
      const minAngle = this.min([xAngle, xAngleAd, yAngle, yAngleAd, zAngle, zAngleAd])// 最小夹角

      switch (minAngle) {
        case xAngle:
          direction = 0// 向x轴正方向旋转90度（还要区分是绕z轴还是绕y轴）
          if (this.normalize.equals(yLine))
            direction = direction + 0.1// 绕z轴顺时针

          else if (this.normalize.equals(yLineAd))
            direction = direction + 0.2// 绕z轴逆时针

          else if (this.normalize.equals(zLine))
            direction = direction + 0.3// 绕y轴逆时针

          else
            direction = direction + 0.4// 绕y轴顺时针

          break
        case xAngleAd:
          direction = 1// 向x轴反方向旋转90度
          if (this.normalize.equals(yLine))
            direction = direction + 0.1// 绕z轴逆时针

          else if (this.normalize.equals(yLineAd))
            direction = direction + 0.2// 绕z轴顺时针

          else if (this.normalize.equals(zLine))
            direction = direction + 0.3// 绕y轴顺时针

          else
            direction = direction + 0.4// 绕y轴逆时针

          break
        case yAngle:
          direction = 2// 向y轴正方向旋转90度
          if (this.normalize.equals(zLine))
            direction = direction + 0.1// 绕x轴逆时针

          else if (this.normalize.equals(zLineAd))
            direction = direction + 0.2// 绕x轴顺时针

          else if (this.normalize.equals(xLine))
            direction = direction + 0.3// 绕z轴逆时针

          else
            direction = direction + 0.4// 绕z轴顺时针

          break
        case yAngleAd:
          direction = 3// 向y轴反方向旋转90度
          if (this.normalize.equals(zLine))
            direction = direction + 0.1// 绕x轴顺时针

          else if (this.normalize.equals(zLineAd))
            direction = direction + 0.2// 绕x轴逆时针

          else if (this.normalize.equals(xLine))
            direction = direction + 0.3// 绕z轴顺时针

          else
            direction = direction + 0.4// 绕z轴逆时针

          break
        case zAngle:
          direction = 4// 向z轴正方向旋转90度
          if (this.normalize.equals(yLine))
            direction = direction + 0.1// 绕x轴顺时针

          else if (this.normalize.equals(yLineAd))
            direction = direction + 0.2// 绕x轴逆时针

          else if (this.normalize.equals(xLine))
            direction = direction + 0.3// 绕y轴顺时针

          else
            direction = direction + 0.4// 绕y轴逆时针

          break
        case zAngleAd:
          direction = 5// 向z轴反方向旋转90度
          if (this.normalize.equals(yLine))
            direction = direction + 0.1// 绕x轴逆时针

          else if (this.normalize.equals(yLineAd))
            direction = direction + 0.2// 绕x轴顺时针

          else if (this.normalize.equals(xLine))
            direction = direction + 0.3// 绕y轴逆时针

          else
            direction = direction + 0.4// 绕y轴顺时针

          break
        default:
          break
      }
      return direction
    },

    // 开始操作魔方
    startCube(event) {
      this.getIntersects(event)
      // 魔方没有处于转动过程中且存在碰撞物体
      if (!this.isRotating && this.intersect) {
        this.startPoint = this.intersect.point// 开始转动，设置起始点
        this.controls.enabled = false// 当刚开始的接触点在魔方上时操作为转动魔方，屏蔽控制器转动
      }
      else {
        this.controls.enabled = true// 当刚开始的接触点没有在魔方上或者在魔方上但是魔方正在转动时操作转动控制器
      }
    },
    // 魔方操作结束
    stopCube() {
      this.intersect = null
      this.startPoint = null
    },
    // 滑动操作魔方
    moveCube(event) {
      this.getIntersects(event)
      if (this.intersect) {
        if (!this.isRotating && this.startPoint) { // 魔方没有进行转动且满足进行转动的条件
          this.movePoint = this.intersect.point
          if (!this.movePoint.equals(this.startPoint)) { // 和起始点不一样则意味着可以得到转动向量了
            this.isRotating = true// 转动标识置为true
            const sub = this.movePoint.sub(this.startPoint)// 计算转动向量
            const direction = this.getDirection(sub)// 获得方向
            const elements = this.getBoxs(this.intersect, direction)
            const startTime = new Date().getTime()
            window.requestAnimFrame((timestamp) => {
              this.rotateAnimation(elements, direction, timestamp, 0)
            })
          }
        }
      }
      event.preventDefault()
    },
    // 绕着世界坐标系的某个轴旋转
    rotateAroundWorldY(obj, rad) {
      const x0 = obj.position.x
      const z0 = obj.position.z
      /**
       * 因为物体本身的坐标系是随着物体的变化而变化的，
       * 所以如果使用rotateZ、rotateY、rotateX等方法，
       * 多次调用后就会出问题，先改为Quaternion实现。
       */
      const q = new THREE.Quaternion()
      q.setFromAxisAngle(new THREE.Vector3(0, 1, 0), rad)
      obj.quaternion.premultiply(q)
      // obj.rotateY(rad);
      obj.position.x = Math.cos(rad) * x0 + Math.sin(rad) * z0
      obj.position.z = Math.cos(rad) * z0 - Math.sin(rad) * x0
    },
    rotateAroundWorldZ(obj, rad) {
      const x0 = obj.position.x
      const y0 = obj.position.y
      const q = new THREE.Quaternion()
      q.setFromAxisAngle(new THREE.Vector3(0, 0, 1), rad)
      obj.quaternion.premultiply(q)
      // obj.rotateZ(rad);
      obj.position.x = Math.cos(rad) * x0 - Math.sin(rad) * y0
      obj.position.y = Math.cos(rad) * y0 + Math.sin(rad) * x0
    },

    rotateAroundWorldX(obj, rad) {
      const y0 = obj.position.y
      const z0 = obj.position.z
      const q = new THREE.Quaternion()
      q.setFromAxisAngle(new THREE.Vector3(1, 0, 0), rad)
      obj.quaternion.premultiply(q)
      // obj.rotateX(rad);
      obj.position.y = Math.cos(rad) * y0 - Math.sin(rad) * z0
      obj.position.z = Math.cos(rad) * z0 + Math.sin(rad) * y0
    },

    /**
     * 旋转动画
     */
    rotateAnimation(elements, direction, currentstamp, startstamp, laststamp) {
      const totalTime = 500// 转动的总运动时间
      if (startstamp === 0) {
        startstamp = currentstamp
        laststamp = currentstamp
      }
      if (currentstamp - startstamp >= totalTime) {
        currentstamp = startstamp + totalTime
        this.isRotating = false
        this.startPoint = null
        this.updateCubeIndex(elements)
      }
      switch (direction) {
        // 绕z轴顺时针
        case 0.1:
        case 1.2:
        case 2.4:
        case 3.3:
          for (let i = 0; i < elements.length; i++)
            this.rotateAroundWorldZ(elements[i], -90 * Math.PI / 180 * (currentstamp - laststamp) / totalTime)

          break
          // 绕z轴逆时针
        case 0.2:
        case 1.1:
        case 2.3:
        case 3.4:
          for (let i = 0; i < elements.length; i++)
            this.rotateAroundWorldZ(elements[i], 90 * Math.PI / 180 * (currentstamp - laststamp) / totalTime)

          break
          // 绕y轴顺时针
        case 0.4:
        case 1.3:
        case 4.3:
        case 5.4:
          for (let i = 0; i < elements.length; i++)
            this.rotateAroundWorldY(elements[i], -90 * Math.PI / 180 * (currentstamp - laststamp) / totalTime)

          break
          // 绕y轴逆时针
        case 1.4:
        case 0.3:
        case 4.4:
        case 5.3:
          for (let i = 0; i < elements.length; i++)
            this.rotateAroundWorldY(elements[i], 90 * Math.PI / 180 * (currentstamp - laststamp) / totalTime)

          break
          // 绕x轴顺时针
        case 2.2:
        case 3.1:
        case 4.1:
        case 5.2:
          for (let i = 0; i < elements.length; i++)
            this.rotateAroundWorldX(elements[i], 90 * Math.PI / 180 * (currentstamp - laststamp) / totalTime)

          break
          // 绕x轴逆时针
        case 2.1:
        case 3.2:
        case 4.2:
        case 5.1:
          for (let i = 0; i < elements.length; i++)
            this.rotateAroundWorldX(elements[i], -90 * Math.PI / 180 * (currentstamp - laststamp) / totalTime)

          break
        default:
          break
      }
      if (currentstamp - startstamp < totalTime) {
        window.requestAnimFrame((timestamp) => {
          this.rotateAnimation(elements, direction, timestamp, startstamp, currentstamp)
        })
      }
    },

    // 更新位置索引
    updateCubeIndex(elements) {
      for (let i = 0; i < elements.length; i++) {
        const temp1 = elements[i]
        for (let j = 0; j < this.initStatus.length; j++) {
          const temp2 = this.initStatus[j]
          if (Math.abs(temp1.position.x - temp2.x) <= cubeParams.len / 2
              && Math.abs(temp1.position.y - temp2.y) <= cubeParams.len / 2
              && Math.abs(temp1.position.z - temp2.z) <= cubeParams.len / 2) {
            temp1.cubeIndex = temp2.cubeIndex
            break
          }
        }
      }
    },

    // 根据方向获得运动元素
    getBoxs(target, direction) {
      let targetId = target.object.cubeIndex
      const ids = []
      for (let i = 0; i < cubes.length; i++)
        ids.push(cubes[i].cubeIndex)

      const minId = this.min(ids)
      targetId = targetId - minId
      const numI = parseInt(targetId / 9)
      const numJ = targetId % 9
      const boxs = []
      // 根据绘制时的规律判断 no = i*9+j
      switch (direction) {
        // 绕z轴
        case 0.1:
        case 0.2:
        case 1.1:
        case 1.2:
        case 2.3:
        case 2.4:
        case 3.3:
        case 3.4:
          for (let i = 0; i < cubes.length; i++) {
            const tempId = cubes[i].cubeIndex - minId
            if (numI === parseInt(tempId / 9))
              boxs.push(cubes[i])
          }
          break
          // 绕y轴
        case 0.3:
        case 0.4:
        case 1.3:
        case 1.4:
        case 4.3:
        case 4.4:
        case 5.3:
        case 5.4:
          for (let i = 0; i < cubes.length; i++) {
            const tempId = cubes[i].cubeIndex - minId
            if (parseInt(numJ / 3) === parseInt(tempId % 9 / 3))
              boxs.push(cubes[i])
          }
          break
          // 绕x轴
        case 2.1:
        case 2.2:
        case 3.1:
        case 3.2:
        case 4.1:
        case 4.2:
        case 5.1:
        case 5.2:
          for (let i = 0; i < cubes.length; i++) {
            const tempId = cubes[i].cubeIndex - minId
            if (tempId % 9 % 3 === numJ % 3)
              boxs.push(cubes[i])
          }
          break
        default:
          break
      }
      return boxs
    },

    // 根据页面宽度和高度创建渲染器，并添加容器中
    initThree() {
      this.width = window.innerWidth
      this.height = window.innerHeight
      this.renderer = new THREE.WebGLRenderer()
      this.renderer.setSize(this.width, this.height)
      this.renderer.setClearColor(0x000000, 1.0)
      document.getElementById('three').appendChild(this.renderer.domElement)
    },

    // 创建场景，后续元素需要加入到场景中才会显示出来
    initScene() {
      scene = new THREE.Scene()
      const axesHelper = new THREE.AxesHelper(500)
      scene.add(axesHelper)
    },

    initLight() {
      const light = new THREE.AmbientLight('#ffffff')
      scene.add(light)
    },

    // 创建相机，并设置正方向和中心点
    initCamera() {
      const width = window.innerWidth// 窗口宽度
      const height = window.innerHeight// ���; //窗口高度
      const k = width / height // 窗口宽高比
      const s = 200 // 三维场景显示范围控制系数，系数越大，显示的范围越大
      // 创建相机对象
      this.camera = new THREE.OrthographicCamera(-s * k, s * k, s, -s, 1, 1000)
      this.camera.position.set(500, 300, 200) // 设置相机位置
      this.camera.lookAt(scene.position) // 设置相机方向(指向的场景对象)
    },

    initObject() {
      // 生成魔方小正方体
      cubes = this.SimpleCube(cubeParams.x, cubeParams.y, cubeParams.z, cubeParams.num, cubeParams.len, cubeParams.colors)
      for (let i = 0; i < cubes.length; i++) {
        const item = cubes[i]
        this.initStatus.push({
          x: item.position.x,
          y: item.position.y,
          z: item.position.z,
          cubeIndex: item.id,
        })
        item.cubeIndex = item.id
        scene.add(item)// 并依次加入到场景中
      }

      // 透明正方体
      const cubegeo = new THREE.BoxGeometry(151, 151, 151)
      console.log(cubegeo)
      const cubemat = new THREE.MeshBasicMaterial({ color: '#ffffff', opacity: 0, transparent: true })
      const cube = new THREE.Mesh(cubegeo, cubemat)
      cube.cubeType = 'coverCube'
      scene.add(cube)
    },

    // 渲染
    render() {
      this.renderer.render(scene, this.camera)
      window.requestAnimFrame(this.render)
    },

    // 生成canvas素材
    faces(rgbaColor) {
      const canvas = document.createElement('canvas')
      canvas.width = 256
      canvas.height = 256
      const context = canvas.getContext('2d')
      if (context) {
        // 画一个宽高都是256的黑色正方形
        context.fillStyle = 'rgba(0,0,0,1)'
        context.fillRect(0, 0, 256, 256)
        // 在内部用某颜色的16px宽的线再画一个宽高为224的圆角正方形并用改颜色填充
        context.rect(20, 20, 224, 224)
        context.lineJoin = 'round'
        context.lineWidth = 20
        context.fillStyle = rgbaColor
        context.strokeStyle = rgbaColor
        context.stroke()
        context.fill()
      }
      else {
        alert('您的浏览器不支持Canvas无法预览.\n')
      }
      return canvas
    },

    SimpleCube(x, y, z, num, len, colors) {
      // 魔方左上角坐标
      const leftUpX = x - num / 2 * len
      const leftUpY = y + num / 2 * len
      const leftUpZ = z + num / 2 * len
      // 根据颜色生成材质
      const materialArr = []
      for (const color of colors) {
        const texture = new THREE.Texture(this.faces(color))
        texture.needsUpdate = true
        const material = new THREE.MeshLambertMaterial({ map: texture })
        materialArr.push(material)
      }
      const cubes = []
      for (let i = 0; i < num; i++) {
        for (let j = 0; j < num * num; j++) {
          const cubegeo = new THREE.BoxGeometry(len, len, len)
          const cube = new THREE.Mesh(cubegeo, materialArr)
          // 依次计算各个小方块中心点坐标
          cube.position.x = (leftUpX + len / 2) + (j % num) * len
          cube.position.y = (leftUpY - len / 2) - parseInt(j / num) * len
          cube.position.z = (leftUpZ - len / 2) - i * len
          cubes.push(cube)
        }
      }
      return cubes
    },
  },
}
</script>

<style>

</style>
