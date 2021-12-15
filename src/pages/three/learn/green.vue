<template>
  <div id="info">
  </div>
</template>
<script>
import * as THREE from 'three'
let renderer, camera, scene, spotLight, mesh
export default {
  name: 'Green',
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
      spotLight.position.set(-50, 50, 0)
      // 渲染阴影
      spotLight.castShadow = true
      scene.add(spotLight)
      const dirLight = new THREE.DirectionalLight(0xFFFFFF)
      dirLight.position.set(30, 10, -50)
      scene.add(dirLight)
      scene.add(new THREE.AmbientLight(0x444444))
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
    init() {
      this.initScene()
      this.initMesh()
      this.initPlane()
      this.initLight()
      this.initCamera()
      // 渲染到页面
      renderer = new THREE.WebGLRenderer()
      renderer.setSize(window.innerWidth, window.innerHeight)
      // 支持渲染阴影
      renderer.shadowMap.enabled = true
      renderer.setClearColor(new THREE.Color(0xFFFFFF))
      this.render()
      const container = document.getElementById('info')
      container.appendChild(renderer.domElement)
    },
    render() {
      requestAnimationFrame(this.render)
      renderer.render(scene, camera)
    },
  },
}
</script>
