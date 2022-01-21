<template>
  <div id="threeBox">
  </div>
</template>

<script setup >
import * as THREE from 'three'
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader'
import { onMounted } from 'vue-demi'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls'

const shipGlbUrl = 'https://static9.xesimg.com/solar-public/public/assets/ship/ship2.glb'

let indexTag = 0
const render = (renderer, scene, camera, points, meshObj) => {
  renderer.render(scene, camera)
  window.requestAnimationFrame(render.bind(null, renderer, scene, camera, points, meshObj))
  const length = points.length
  let subVector
  if (indexTag < (length - 1)) {
    const tag = ++indexTag
    // subVector = points[tag + 1000].sub(points[tag])
  }
  else {
    indexTag = 0
    subVector = points[indexTag]
  }
  // const up = meshObj.up
  // const q = new THREE.Quaternion()
  // const rad = up.angleTo(subVector)
  // q.setFromAxisAngle(new THREE.Vector3(0, 0, 1), rad)
  // meshObj.quaternion.premultiply(q)
  meshObj.position.set(points[indexTag].x, points[indexTag].y, points[indexTag].z)
}

const randomPoint = (tag) => {
  const result = Math.floor(Math.random() * tag)
  if (Math.random() < 0.5)
    return 0 - result

  return result
}

const init = async() => {
  const scene = new THREE.Scene()
  // 参考地址 https://threejs.org/docs/index.html?q=GLTFLoader#examples/zh/loaders/GLTFLoader
  const loader = new GLTFLoader()
  // 坐标系
  const helper = new THREE.AxesHelper(1000)
  scene.add(helper)
  // 加载模型
  const { scene: shipObj } = await loader.loadAsync(shipGlbUrl)
  shipObj.scale.set(15, 15, 15)
  scene.add(shipObj)
  // 轨迹
  const geometry = new THREE.BufferGeometry() // 声明一个几何体对象Geometry
  // 随机生成三维样条曲线
  const randomTag = 400
  const pointsCount = 50
  const pointList = []
  for (let i = 0; i < pointsCount; i++) {
    const x = randomPoint(randomTag)
    const y = randomPoint(randomTag)
    const z = randomPoint(randomTag)
    pointList.push(new THREE.Vector3(x, y, z))
  }
  const curve = new THREE.CatmullRomCurve3(pointList)
  // getPoints是基类Curve的方法，返回一个vector3对象作为元素组成的数组
  // 分段越多点约密集
  const points = curve.getPoints(10000) // 分段数100，返回101个顶点
  // setFromPoints方法从points中提取数据改变几何体的顶点属性vertices
  geometry.setFromPoints(points)
  // 材质对象
  const material = new THREE.LineBasicMaterial({
    color: 'red',
    side: THREE.DoubleSide,
  })
  // 线条模型对象
  const line = new THREE.Line(geometry, material)
  scene.add(line)
  // 创建相机
  const width = window.innerWidth
  const height = window.innerHeight
  const k = width / height
  const s = 200
  // 创建相机对象
  const camera = new THREE.OrthographicCamera(-s * k, s * k, s, -s, 0.1, 1000)
  camera.position.set(0, 200, 500)
  camera.lookAt(scene.position)
  const renderer = new THREE.WebGLRenderer({
    antialias: true,
  })
  // 启用 shebeifenbianlv
  // https://developer.mozilla.org/zh-CN/docs/Web/API/Window/devicePixelRatio
  // https://threejs.org/docs/index.html?q=webGL#api/zh/renderers/WebGLRenderer
  renderer.setPixelRatio(window.devicePixelRatio)
  renderer.setSize(width, height)
  renderer.setClearColor(0xFFFFFF)
  render(renderer, scene, camera, points, shipObj)
  const container = document.getElementById('threeBox')
  const controls = new OrbitControls(camera, renderer.domElement)// 创建控件对象
  // controls.addEventListener('change', render.bind(null, renderer, scene, camera))// 监听鼠标、键盘事件
  container.appendChild(renderer.domElement)
}
onMounted(() => {
  init()
})
</script>

<style scoped>
#threeBox {
  position: absolute;
  left: 0;
  right: 0;
  bottom: 0;
  top: 0;
}
</style>
