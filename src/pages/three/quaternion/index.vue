<template>
  <div id="info">
  </div>
</template>

<script>

import * as THREE from 'three'
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
      const angle = Math.PI / 2
      q.setFromAxisAngle(new THREE.Vector3(0, 1, 0), Math.PI / 2)
      cube.quaternion.premultiply(q)
      // // obj.rotateY(rad);
      // cube.position.x = Math.cos(angle) * cube.position.x - Math.sin(angle) * cube.position.y
      // cube.position.y = Math.cos(angle) * cube.position.y + Math.sin(angle) * cube.position.x
      cube.quaternion.slerp(q, 0.2)
      renderer.render(scene, camera)
    },
  },
}
</script>

<style scoped>

</style>
