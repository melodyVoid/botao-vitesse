<template>
  <div id="container">
  </div>
</template>

<script>
import * as THREE from 'three'
import { GUI } from 'three/examples/jsm/libs/dat.gui.module'
let container
let camera, scene, renderer
let raycaster, pointer
let mesh, line
const controls = {
  x: 0.1,
  y: 0.2,
}
const gui = new GUI()
gui.add(controls, 'x', 0, Math.PI)
gui.add(controls, 'y', 0, Math.PI)
export default {
  name: 'BufferGeometry',
  mounted() {
    this.init()
    this.animate()
  },
  methods: {
    init() {
      container = document.getElementById('container')
      camera = new THREE.PerspectiveCamera(27, window.innerWidth / window.innerHeight, 1, 3500)
      camera.position.z = 2800
      scene = new THREE.Scene()
      console.log(scene)
      scene.background = new THREE.Color(0x050505)
      scene.fog = new THREE.Fog(0x050505, 2000, 3500)
      // 环境光会均匀的照亮场景中的所有物体。环境光不能用来投射阴影，因为它没有方向。
      scene.add(new THREE.AmbientLight(0x444444))
      // 平行光是沿着特定方向发射的光。这种光的表现像是无限远,从它发出的光线都是平行的。常常用平行光来模拟太阳光 的效果; 太阳足够远，因此我们可以认为太阳的位置是无限远，所以我们认为从太阳发出的光线也都是平行的。
      const light1 = new THREE.DirectionalLight(0xFFFFFF, 0.5)
      light1.position.set(1, 1, 1)
      const light2 = new THREE.DirectionalLight(0xFFFFFF, 1.5)
      light2.position.set(0, -1, 0)
      scene.add(light2)
      // 个数
      const triangles = 5000
      let geometry = new THREE.BufferGeometry()
      const positions = new Float32Array(triangles * 3 * 3)
      const normals = new Float32Array(triangles * 3 * 3)
      const colors = new Float32Array(triangles * 3 * 3)
      const color = new THREE.Color()
      const n = 800; const n2 = n / 2
      const d = 120; const d2 = d / 2
      const pA = new THREE.Vector3()
      const pB = new THREE.Vector3()
      const pC = new THREE.Vector3()
      const cb = new THREE.Vector3()
      const ab = new THREE.Vector3()
      for (let i = 0; i < positions.length; i += 9) {
        // // positions
        const x = Math.random() * n - n2
        const y = Math.random() * n - n2
        const z = Math.random() * n - n2
        const ax = x + Math.random() * d - d2
        const ay = y + Math.random() * d - d2
        const az = z + Math.random() * d - d2
        const bx = x + Math.random() * d - d2
        const by = y + Math.random() * d - d2
        const bz = z + Math.random() * d - d2
        const cx = x + Math.random() * d - d2
        const cy = y + Math.random() * d - d2
        const cz = z + Math.random() * d - d2
        positions[i] = ax
        positions[i + 1] = ay
        positions[i + 2] = az
        positions[i + 3] = bx
        positions[i + 4] = by
        positions[i + 5] = bz
        positions[i + 6] = cx
        positions[i + 7] = cy
        positions[i + 8] = cz

        // flat face normals
        pA.set(ax, ay, az)
        pB.set(bx, by, bz)
        pC.set(cx, cy, cz)
        cb.subVectors(pC, pB)
        ab.subVectors(pA, pB)
        cb.cross(ab)
        cb.normalize()
        const nx = cb.x
        const ny = cb.y
        const nz = cb.z

        normals[i] = nx
        normals[i + 1] = ny
        normals[i + 2] = nz

        normals[i + 3] = nx
        normals[i + 4] = ny
        normals[i + 5] = nz

        normals[i + 6] = nx
        normals[i + 7] = ny
        normals[i + 8] = nz
        // Color
        const vx = (x / n) + 0.5
        const vy = (y / n) + 0.5
        const vz = (z / n) + 0.5

        color.setRGB(vx, vy, vz)

        colors[i] = color.r
        colors[i + 1] = color.g
        colors[i + 2] = color.b

        colors[i + 3] = color.r
        colors[i + 4] = color.g
        colors[i + 5] = color.b

        colors[i + 6] = color.r
        colors[i + 7] = color.g
        colors[i + 8] = color.b
      }
      geometry.setAttribute('position', new THREE.BufferAttribute(positions, 3))
      geometry.setAttribute('normal', new THREE.BufferAttribute(normals, 3))
      geometry.setAttribute('color', new THREE.BufferAttribute(colors, 3))
      // https://threejs.org/docs/index.html#api/zh/core/BufferGeometry.computeBoundingSphere
      geometry.computeBoundingSphere()
      let material = new THREE.MeshPhongMaterial({
        color: 0xAAAAAA,
        specular: 0xFFFFFF,
        shininess: 250,
        side: THREE.DoubleSide,
        vertexColors: true,
      })
      mesh = new THREE.Mesh(geometry, material)
      scene.add(mesh)
      raycaster = new THREE.Raycaster()
      pointer = new THREE.Vector2()
      geometry = new THREE.BufferGeometry()
      geometry.setAttribute('position', new THREE.BufferAttribute(new Float32Array(4 * 3), 3))
      material = new THREE.LineBasicMaterial({ color: 0xFFFFFF, transparent: true })
      line = new THREE.Line(geometry, material)
      scene.add(line)

      renderer = new THREE.WebGLRenderer()
      renderer.setPixelRatio(window.devicePixelRatio)
      renderer.setSize(window.innerWidth, window.innerHeight)
      container.appendChild(renderer.domElement)
      window.addEventListener('resize', this.onWindowResize)
      window.addEventListener('pointermove', this.onPointerMove)
    },
    onWindowResize() {
      camera.aspect = window.innerWidth / window.innerHeight
      camera.updateProjectionMatrix()

      renderer.setSize(window.innerWidth, window.innerHeight)
    },
    onPointerMove(event) {
      // https://juejin.cn/post/7021519175898103845
      const canves = document.getElementById('container')
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
    animate() {
      requestAnimationFrame(this.animate)
      this.render()
    },
    render() {
      // const time = Date.now() * 0.001

      mesh.rotation.x = controls.x
      mesh.rotation.y = controls.y
      raycaster.setFromCamera(pointer, camera)
      const intersects = raycaster.intersectObject(mesh)
      if (intersects.length > 0) {
        const intersect = intersects[0]
        const face = intersect.face
        console.log(face)
        const linePosition = line.geometry.attributes.position
        const meshPosition = mesh.geometry.attributes.position
        linePosition.copyAt(0, meshPosition, face.a)
        linePosition.copyAt(1, meshPosition, face.b)
        linePosition.copyAt(2, meshPosition, face.c)
        linePosition.copyAt(3, meshPosition, face.a)
        console.log(linePosition)
        mesh.updateMatrix()
        line.geometry.applyMatrix4(mesh.matrix)
        line.visible = true
      }
      else {
        line.visible = false
      }
      renderer.render(scene, camera)
    },

  },
}
</script>

<style scoped>

</style>
