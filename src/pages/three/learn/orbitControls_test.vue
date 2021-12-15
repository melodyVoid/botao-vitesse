<template>
  <div id="info">
  </div>
</template>
<script>
import * as THREE from 'three'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls'
let renderer, camera, scene, spotLight, mesh
export default {
  name: 'OrbitControlsTest',
  mounted() {
    this.init()
  },
  methods: {
    // 创建场景
    initScene() {
      // 创建场景
      scene = new THREE.Scene()
      const axesHelper = new THREE.AxesHelper(500)
      scene.add(axesHelper)
    },
    // 初始化物体模型
    initMesh() {
      // 创建几何体
      const geometry = new THREE.BoxGeometry(40, 40, 40)
      // 创建材质
      const loader1 = new THREE.TextureLoader()
      const texture1 = loader1.load('https://s3.bmp.ovh/imgs/2021/11/066a3e0b4ed22755.jpeg')
      const material = new THREE.MeshPhongMaterial({
        map: texture1,
      })
      // 创建模型
      mesh = new THREE.Mesh(geometry, material)
      // 投射阴影
      mesh.castShadow = true
      mesh.position.y = 20
      scene.add(mesh)
    },
    // 初始化平面
    initPlane() {
      // 接受阴影
      const planeGeometry = new THREE.PlaneGeometry(600, 200)
      // 初始化一个加载器
      const loader = new THREE.TextureLoader()
      const texture = loader.load('https://s3.bmp.ovh/imgs/2021/11/11724f955fb17e1e.jpg')
      texture.wrapS = THREE.RepeatWrapping
      texture.wrapT = THREE.RepeatWrapping
      // uv两个方向纹理重复数量
      texture.repeat.set(2, 2)
      const planeMaterial = new THREE.MeshLambertMaterial({ map: texture })
      const plane = new THREE.Mesh(planeGeometry, planeMaterial)
      plane.rotation.x = -0.5 * Math.PI
      plane.position.set(15, 0, 0)
      // 接受阴影
      plane.receiveShadow = true
      scene.add(plane)
    },
    // 初始化光源
    initLight() {
      // // 创建光源
      spotLight = new THREE.SpotLight(0xFFFFFF)
      spotLight.position.set(-70, 60, 0)
      spotLight.castShadow = true
      scene.add(new THREE.AmbientLight(0x343434))
      scene.add(spotLight)
    },
    // 初始化相机
    initCamera() {
      // 创建相机
      const width = window.innerWidth
      const height = window.innerHeight
      const k = width / height
      const s = 200
      // 创建相机对象
      camera = new THREE.OrthographicCamera(-s * k, s * k, s, -s, 1, 1000)
      // const camera = new THREE.PerspectiveCamera(45, k, 1, 1000)
      camera.position.set(200, 100, 50)
      camera.lookAt(scene.position)
    },
    init() {
      this.initScene()
      this.initMesh()
      this.initPlane()
      this.initLight()
      this.initCamera()
      // 渲染到页面
      renderer = new THREE.WebGLRenderer()
      renderer.setSize(window.innerWidth, window.innerHeight)
      renderer.shadowMap.enabled = true
      renderer.setClearColor(new THREE.Color(0xFFFFFF))
      const container = document.getElementById('info')
      container.appendChild(renderer.domElement)
      this.render()
      const controls = new OrbitControls(camera, renderer.domElement)// 创建控件对象
      controls.addEventListener('change', this.render)// 监听鼠标、键盘事件
    },
    render() {
      renderer.render(scene, camera)
    },
  },
}
</script>
