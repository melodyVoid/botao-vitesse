<template>
  <div id="info">
  </div>
</template>
<script>
import * as THREE from 'three'
let scene, spotLight, mesh, camera
export default {
  name: 'Light',
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
      // 投射阴影
      mesh.castShadow = true
      scene.add(mesh)
    },
    // 初始化平面
    initPlane() {
      // 接受阴影
      const planeGeometry = new THREE.PlaneGeometry(600, 200)
      const planeMaterial = new THREE.MeshLambertMaterial({ color: 0xAAAAAA })
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
      spotLight.position.set(-50, 50, 0)
      // 渲染阴影
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
      camera.position.set(500, 300, 200)
      camera.lookAt(scene.position)
    },
    // 初始化方法
    init() {
      this.initScene()
      this.initMesh()
      this.initPlane()
      this.initLight()
      this.initCamera()
      // 渲染到页面
      const renderer = new THREE.WebGLRenderer()
      renderer.setSize(window.innerWidth, window.innerHeight)
      // 支持渲染阴影
      renderer.shadowMap.enabled = true
      renderer.setClearColor(new THREE.Color(0xFFFFFF))
      renderer.render(scene, camera)
      const container = document.getElementById('info')
      container.appendChild(renderer.domElement)
    },
  },
}
</script>
<style scoped>
</style>
