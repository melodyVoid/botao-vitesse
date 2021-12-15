<template>
  <div id="info">
  </div>
</template>
<script>
import * as THREE from 'three'
export default {
  name: 'First',
  mounted() {
    this.init()
  },
  methods: {
    init() {
      // 创建场景
      const scene = new THREE.Scene()
      // 创建几何体
      const geometry = new THREE.BoxGeometry(100, 100, 100)
      // 创建材质
      const material = new THREE.MeshPhongMaterial({
        color: '#ffffff',
      })
      // 创建模型
      const mesh = new THREE.Mesh(geometry, material)
      scene.add(mesh)
      // 创建光源
      const point = new THREE.PointLight('#ffffff')
      point.position.set(400, 200, 300)
      scene.add(point)
      // 创建相机
      const width = window.innerWidth
      const height = window.innerHeight
      const k = width / height
      const s = 200
      // 创建相机对象
      const camera = new THREE.OrthographicCamera(-s * k, s * k, s, -s, 1, 1000)
      camera.position.set(500, 300, 200)
      camera.lookAt(scene.position)
      // 渲染到页面
      const renderer = new THREE.WebGLRenderer()
      renderer.setSize(window.innerWidth, window.innerHeight)
      renderer.setClearColor(0x999999)
      renderer.render(scene, camera)
      const container = document.getElementById('info')
      container.appendChild(renderer.domElement)
    },
  },
}
</script>
