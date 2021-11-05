<template>
  <div>
  </div>
</template>

<script>
import * as THREE from 'three'
export default {
  name: 'Index',
  mounted() {
    const vecx = new THREE.Vector3(1, 0, 0)
    const vecy = new THREE.Vector3(0, 1, 0)
    const vecz = new THREE.Vector3(0, 0, 1)
    const angle1 = vecx.angleTo(vecy)
    const angle2 = vecy.angleTo(vecz)
    console.log(angle1)
    console.log(angle2)
    console.log(angle2 === Math.PI/2)
    console.log(vecx - vecy)
    console.log(vecx.sub(vecy));
  },
}
</script>

<style scoped>

</style>
