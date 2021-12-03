<template>
  <div id="sun-system">
  </div>
</template>

<script>
import { defineComponent } from 'vue'
import { GUI } from 'three/examples/jsm/libs/dat.gui.module'
import * as THREE from 'three'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls'

let scene, camera, renderer
const s = 1000 // 三维场景显示范围控制系数，系数越大，显示的范围越大
const width = window.innerWidth // 窗口宽度
const height = window.innerHeight // 窗口高度
const k = width / height // 窗口宽高比
let sun // 太阳对象
let earth // 地球

const controls = {
  太阳旋转速度: 0.02,
  地球旋转速度: 0.02,
  地球公转速度: 0.02,
}
const gui = new GUI()
gui.add(controls, '太阳旋转速度', 0, 0.2)
gui.add(controls, '地球旋转速度', 0, 0.2)
gui.add(controls, '地球公转速度', 0, 0.2)
export default defineComponent(
  {
    name: 'Index',
    mounted() {
      this.init()
      const v1 = new THREE.Vector3(0, 1, 0)
      const v2 = new THREE.Vector3(Math.sin(Math.PI / 4), Math.cos(Math.PI / 4), 0)
      const v3 = new THREE.Vector3(1, 1, 0).normalize()
      console.log(v1.angleTo(v3))
      console.log(v3)
    },
    methods: {
      init() {
        const loader = new THREE.TextureLoader()
        const cubeLoader = new THREE.CubeTextureLoader()
        // 场景
        scene = new THREE.Scene()
        cubeLoader.setPath('/background/')
        scene.background = cubeLoader.load([
          'right.png', 'left.png',
          'top.png', 'bottom.png',
          'front.png', 'back.png',
        ])
        // camera = new THREE.OrthographicCamera(-s * k, s * k, s, -s, 0.1, 1000)
        camera = new THREE.PerspectiveCamera(45, k, 0.1, 1000)
        // 辅助坐标系
        const axes = new THREE.AxesHelper(500)
        scene.add(axes)
        camera.position.set(0, 0, 800)
        camera.lookAt(scene.position)
        /**
         * 太阳
         * @type {SphereGeometry}
         */
        const sunGeometry = new THREE.SphereGeometry(100, 32, 32)
        const sunTexture = loader.load('/stars/sun.png')
        const sunMaterial = new THREE.MeshLambertMaterial({ map: sunTexture })
        sun = new THREE.Mesh(sunGeometry, sunMaterial)
        scene.add(sun)
        /**
         * 地球
         * @type {SphereGeometry}
         */
        const earthGeometry = new THREE.SphereGeometry(20, 32, 32)
        const earthTexture = loader.load('/stars/earth/earth_diffuse.png')
        const earthSpecularTexture = loader.load('/stars/earth/earth_specular.png')
        const normalTexture = loader.load('/stars/earth/earth_normal.png')
        const earthMaterial = new THREE.MeshLambertMaterial({
          map: earthTexture,
          specularMap: earthSpecularTexture,
          normalMap: normalTexture,
          normalScale: new THREE.Vector2(3, 3),
          shininess: 30, // 高光部分的亮度，默认30
          specular: 0xFF0000, // 高光部分的颜色
        })
        earth = new THREE.Mesh(earthGeometry, earthMaterial)
        earth.position.x = 200
        earth.rotateZ(-(Math.PI / 8))
        scene.add(earth)
        /**
         * 地球轨道线
         * @type {EllipseCurve}
         */
        const curve = new THREE.EllipseCurve(
          0, 0, 200, 200, // xRadius, yRadius
        )

        const curvePoints = curve.getPoints(50)
        const curveGeometry = new THREE.BufferGeometry().setFromPoints(curvePoints)
        const curveMaterial = new THREE.LineBasicMaterial({ color: 0xFFFFFF })
        const ellipse = new THREE.Line(curveGeometry, curveMaterial)
        ellipse.rotateX(Math.PI / 2)
        scene.add(ellipse)
        // console.log(earth.position)
        // 光源
        const ambienLight = new THREE.AmbientLight(0xFFFFFF, 0.8)
        scene.add(ambienLight)
        // 点光源
        const pointLight = new THREE.PointLight(0xFFFFFF)
        pointLight.position.set(0, 0, 0)
        scene.add(pointLight)
        // 渲染器
        renderer = new THREE.WebGLRenderer()
        renderer.setPixelRatio(window.devicePixelRatio)
        renderer.setSize(width, height)
        document.getElementById('sun-system').appendChild(renderer.domElement)
        this.ThreeRender()
        // 视角工具
        const orbitControls = new OrbitControls(camera, renderer.domElement)// 创建控件对象
        orbitControls.addEventListener('change', this.orbitRender)// 监听鼠标、键盘事件
        // 控制台
      },
      orbitRender() {
        renderer.render(scene, camera)
      },
      ThreeRender() {
        // 旋转
        sun.rotateY(controls.太阳旋转速度)
        // 地球自转
        this.earchRotate()
        // 地球绕太阳转
        this.earthFllowSun()
        renderer.render(scene, camera)
        window.requestAnimationFrame(this.ThreeRender)
      },
      // 地球自转
      earchRotate() {
        const q = new THREE.Quaternion()
        q.setFromAxisAngle(new THREE.Vector3(Math.sin(Math.PI / 8), Math.cos(Math.PI / 8), 0), controls.地球旋转速度)
        // q.setFromAxisAngle(new THREE.Vector3(1, 1, 0).normalize(), controls.地球旋转速度)
        earth.quaternion.premultiply(q)
      },
      // 地球公转位置计算
      earthFllowSun() {
        const x0 = earth.position.x
        const z0 = earth.position.z
        const revolution = controls.地球公转速度
        earth.position.x = Math.cos(revolution) * x0 + Math.sin(revolution) * z0
        earth.position.z = Math.cos(revolution) * z0 - Math.sin(revolution) * x0
      },
    },
  },
)
</script>

<style scoped>
#sun-system {
  position: absolute;
  left: 0;
  right: 0;
  bottom: 0;
  top: 0;
}
</style>
