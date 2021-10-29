<template>
  <div>
    <a href="https://github.com/mrdoob/three.js/blob/master/examples/webgl_interactive_cubes.html" target="_blank" rel="noopener">官网代码参考地址</a>
    <div id="info">
    </div>
  </div>
</template>

<script>
import * as THREE from 'three'
let container
let camera, scene, raycaster, renderer

let INTERSECTED
let theta = 0

const pointer = new THREE.Vector2()
const radius = 100

export default {
  name: 'Index',
  mounted() {
    this.init()
    this.animate()
  },
  methods: {
    /**
     * 初始化 scene， light 随机
     */
    init() {
      container = document.getElementById('info')
      // 透视相机
      // camera = new THREE.PerspectiveCamera(70, window.innerWidth / window.innerHeight, 1, 10000)
      // 正交相机
      const width = window.innerWidth// 窗口宽度
      const height = window.innerHeight// ���; //窗口高度
      const k = width / height // 窗口宽高比
      const s = 800 // 三维场景显示范围控制系数，系数越大，显示的范围越大
      // 创建相机对象
      camera = new THREE.OrthographicCamera(-s * k, s * k, s, -s, 1, 1000)

      scene = new THREE.Scene()
      scene.background = new THREE.Color(0xF0F0F0)

      const light = new THREE.DirectionalLight(0xFFFFFF, 1)
      light.position.set(1, 1, 1).normalize()
      scene.add(light)

      const geometry = new THREE.BoxGeometry(20, 20, 20)

      for (let i = 0; i < 2000; i++) {
        const object = new THREE.Mesh(geometry, new THREE.MeshLambertMaterial({ color: Math.random() * 0xFFFFFF }))

        object.position.x = Math.random() * 800 - 400
        object.position.y = Math.random() * 800 - 400
        object.position.z = Math.random() * 800 - 400

        object.rotation.x = Math.random() * 2 * Math.PI
        object.rotation.y = Math.random() * 2 * Math.PI
        object.rotation.z = Math.random() * 2 * Math.PI

        object.scale.x = Math.random() + 0.5
        object.scale.y = Math.random() + 0.5
        object.scale.z = Math.random() + 0.5

        scene.add(object)
      }

      raycaster = new THREE.Raycaster()

      renderer = new THREE.WebGLRenderer()
      renderer.setPixelRatio(window.devicePixelRatio)
      renderer.setSize(window.innerWidth, window.innerHeight)
      container.appendChild(renderer.domElement)
      document.addEventListener('mousemove', this.onPointerMove)

      //

      window.addEventListener('resize', this.onWindowResize)
    },
    animate() {
      requestAnimationFrame(this.animate)
      this.render()
    },

    onWindowResize() {
      camera.aspect = window.innerWidth / window.innerHeight
      camera.updateProjectionMatrix()

      renderer.setSize(window.innerWidth, window.innerHeight)
    },

    /**
     * 监听鼠标移动事件，并设置点
     * document.addEventListener('mousemove', this.onPointerMove)
     * @param event
     */
    onPointerMove(event) {
      // https://juejin.cn/post/7021519175898103845
      const canves = document.getElementById('info')
      const canvesWidth = canves.getBoundingClientRect().width
      const canvesHeight = canves.getBoundingClientRect().height
      const top = canves.getBoundingClientRect().top
      const left = canves.getBoundingClientRect().left
      const right = canves.getBoundingClientRect().right
      // pointer.x = ((event.clientX) / window.innerWidth) * 2 - 1
      // pointer.y = -((event.clientY) / window.innerHeight) * 2 + 1
      // 坐标归一化
      pointer.x = ((event.clientX - (left) - (right - canvesWidth)) / canvesWidth) * 2 - 1
      pointer.y = -((event.clientY - top) / canvesHeight) * 2 + 1
    },
    render() {
      theta += 0.1
      // 旋转起来
      camera.position.x = radius * Math.sin(THREE.MathUtils.degToRad(theta))
      camera.position.y = radius * Math.sin(THREE.MathUtils.degToRad(theta))
      camera.position.z = radius * Math.cos(THREE.MathUtils.degToRad(theta))
      camera.lookAt(scene.position)

      camera.updateMatrixWorld()

      // find intersections

      raycaster.setFromCamera(pointer, camera)

      const intersects = raycaster.intersectObjects(scene.children, false)

      if (intersects.length > 0) {
        // for (const { object } of intersects)
        //   object.material.emissive.setHex(0xFF0000)

        if (INTERSECTED !== intersects[0].object) {
          if (INTERSECTED)
            INTERSECTED.material.emissive.setHex(INTERSECTED.currentHex)

          INTERSECTED = intersects[0].object
          INTERSECTED.currentHex = INTERSECTED.material.emissive.getHex()
          INTERSECTED.material.emissive.setHex(0xFF0000)
        }
      }
      else {
        if (INTERSECTED) INTERSECTED.material.emissive.setHex(INTERSECTED.currentHex)

        INTERSECTED = null
      }

      renderer.render(scene, camera)
    },
  },
}
</script>

<style scoped>
#info{
  width: 100%;
  height: 100%;
}
</style>
