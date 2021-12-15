<template>
  <div id="info">
  </div>
</template>
<script>
import * as THREE from 'three'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls'
import { GUI } from 'three/examples/jsm/libs/dat.gui.module'
let renderer, camera, scene, spotLight, mesh, material
const guiControls = {
  x: 0,
  y: 0,
  z: 0,
  color: 0x7777FF,
}
// dat.gui控制器
const gui = new GUI()
gui.add(guiControls, 'x', 0, 200)
gui.add(guiControls, 'y', 0, 200)
gui.add(guiControls, 'z', 0, 200)
export default {
  name: 'DatGui',
  mounted() {
    this.init()
  },
  methods: {
    // 创建场景
    initScene() {
      scene = new THREE.Scene()
    },
    // 初始化物体模型
    initMesh() {
      // 创建几何体
      const geometry = new THREE.BoxGeometry(40, 40, 40)
      // 创建材质
      const material = new THREE.MeshPhongMaterial({
        color: 0x7777FF,
      })
      // 创建模型
      mesh = new THREE.Mesh(geometry, material)
      mesh.castShadow = true
      scene.add(mesh)
      const axesHelper = new THREE.AxesHelper(500)
      scene.add(axesHelper)
    },
    // 初始化平面
    initPlane() {
      // 阴影接手面
      const planeGeometry = new THREE.PlaneGeometry(600, 200)
      const planeMaterial = new THREE.MeshLambertMaterial({ color: 0xAAAAAA })
      const plane = new THREE.Mesh(planeGeometry, planeMaterial)
      plane.rotation.x = -0.5 * Math.PI
      plane.position.set(15, 0, 0)
      plane.receiveShadow = true
      scene.add(plane)
    },
    // 初始化光源
    initLight() {
      // 创建光源
      spotLight = new THREE.SpotLight(0xFFFFFF)
      spotLight.position.set(-50, 50, 0)
      spotLight.castShadow = true
      scene.add(new THREE.AmbientLight(0x343434))
      scene.add(spotLight)
      const spotLightHelper = new THREE.SpotLightHelper(spotLight)
      scene.add(spotLightHelper)
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
      renderer.setClearColor(new THREE.Color(0x000000))
      const container = document.getElementById('info')
      container.appendChild(renderer.domElement)
      this.render()
      const controls = new OrbitControls(camera, renderer.domElement)// 创建控件对象
      controls.addEventListener('change', this.render)// 监听鼠标、键盘事件
      // 增加控制器
      gui.addColor(guiControls, 'color').onChange((e) => {
        material.color = new THREE.Color(e)
      })
    },
    // 渲染函数
    render() {
      mesh.position.x = guiControls.x
      mesh.position.y = guiControls.y
      mesh.position.z = guiControls.z
      requestAnimationFrame(this.render)
      renderer.render(scene, camera)
    },
  },
}
</script>
