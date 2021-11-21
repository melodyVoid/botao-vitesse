<template>
  <div id="info">
  </div>
</template>

<script>

import * as THREE from 'three'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls'
let renderer, scene, camera, cube
export default {
  name: 'Index',
  mounted() {
    this.init()
    this.animate()
  },
  methods: {

    init() {
      const container = document.getElementById('info')
      scene = new THREE.Scene()
      const axesHelper = new THREE.AxesHelper(500)
      scene.add(axesHelper)
      const geometry = new THREE.BoxGeometry(100, 100, 100)
      const material = new THREE.MeshBasicMaterial({ color: 'red' })
      cube = new THREE.Mesh(geometry, material)
      cube.position.x = 100
      scene.add(cube)
      // 环境光会均匀的照亮场景中的所有物体。环境光不能用来投射阴影，因为它没有方向。
      scene.add(new THREE.AmbientLight(0xFFFFFF))
      // 正交相机
      const width = window.innerWidth// 窗口宽度
      const height = window.innerHeight// ���; //窗口高度
      const k = width / height // 窗口宽高比
      const s = 800 // 三维场景显示范围控制系数，系数越大，显示的范围越大
      // 创建相机对象
      camera = new THREE.OrthographicCamera(-s * k, s * k, s, -s, 1, 1000)
      camera.position.set(200, 300, 200)
      camera.lookAt(scene.position)
      renderer = new THREE.WebGLRenderer()
      renderer.setPixelRatio(window.devicePixelRatio)
      renderer.setSize(window.innerWidth, window.innerHeight)
      container.appendChild(renderer.domElement)
      // renderer.render(scene, camera)
    },
    animate() {
      requestAnimationFrame(this.animate)
      this.render()
    },
    render() {
      const q = new THREE.Quaternion()
      const rad = 0.02
      const x0 = cube.position.x
      const z0 = cube.position.z
      // 自身旋转
      q.setFromAxisAngle(new THREE.Vector3(0, 1, 0), rad)
      cube.quaternion.premultiply(q)
      // 世界左边变换
      cube.position.x = Math.cos(rad) * x0 + Math.sin(rad) * z0
      cube.position.z = Math.cos(rad) * z0 - Math.sin(rad) * x0
      renderer.render(scene, camera)
    },
  },
}
</script>

<style scoped>

</style>
