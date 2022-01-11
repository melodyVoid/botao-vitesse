<template>
  <div id="three">
  </div>
</template>

<script>
import * as THREE from 'three'
import { CSS2DRenderer, CSS2DObject } from 'three/examples/jsm/renderers/CSS2DRenderer'
let scene, camera, light, renderer
export default {
  name: 'Curve',
  mounted() {
    this.init()
  },
  methods: {
    init() {
      scene = new THREE.Scene()
      const axesHelper = new THREE.AxesHelper(500)
      scene.add(axesHelper)
      const curve = new THREE.EllipseCurve(
        0, 0, // ax, aY
        30, 10, // xRadius, yRadius
        0, 2 * Math.PI, // aStartAngle, aEndAngle
        false, // aClockwise
        Math.PI / 4, // aRotation
      )

      const points = curve.getPoints(50)
      const geometry = new THREE.BufferGeometry().setFromPoints(points)

      const material = new THREE.LineBasicMaterial({ color: 'red' })
      const ellipse = new THREE.Line(geometry, material)
      scene.add(ellipse)
      const curve1 = new THREE.CatmullRomCurve3([
        new THREE.Vector3(-10, 0, 10),
        new THREE.Vector3(-5, 5, 5),
        new THREE.Vector3(0, 0, 0),
        new THREE.Vector3(5, -5, 5),
        new THREE.Vector3(10, 0, 10),
      ], true)

      const points1 = curve1.getPoints(50)
      const geometry1 = new THREE.BufferGeometry().setFromPoints(points1)

      const material1 = new THREE.LineBasicMaterial({ color: 'green' })

      // Create the final object to add to the scene
      const curveObject = new THREE.Line(geometry1, material1)
      curveObject.position.set(20, 0, 0)
      scene.add(curveObject)

      // css2DRender
      const earthDiv = document.createElement('div')
      earthDiv.innerHTML = '这是一个标签'
      earthDiv.style.cssText = 'font-size: 12px; color: red;width:50px'
      const earthLabel = new CSS2DObject(earthDiv)
      earthLabel.position.set(30, 0, 0)
      scene.add(earthLabel)

      light = new THREE.AmbientLight(0xFFFFFF)
      scene.add(light)
      const width = window.innerWidth
      const height = window.innerHeight
      const k = width / height
      const s = 200
      // 创建相机对象
      camera = new THREE.OrthographicCamera(-s * k, s * k, s, -s, 1, 1000)
      camera.position.set(0, 10, 30)
      camera.lookAt(scene.position)
      renderer = new THREE.WebGLRenderer()
      renderer.setSize(window.innerWidth, window.innerHeight)
      renderer.setClearColor(0xCDDDCC)
      const container = document.getElementById('three')
      container.appendChild(renderer.domElement)

      const labelRenderer = new CSS2DRenderer()
      labelRenderer.setSize(window.innerWidth, window.innerHeight)
      labelRenderer.domElement.style.cssText = 'position: absolute; top: 0;'
      container.appendChild(labelRenderer.domElement)
      renderer.render(scene, camera)
      labelRenderer.render(scene, camera)
    },
  },
}
</script>

<style scoped>
#three{
  position: fixed;
  top: 0;
  left: 0;
}
</style>
