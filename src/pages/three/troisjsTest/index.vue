<template>
  <Renderer :width="width" :height="height">
    <Camera :position="{ z: 10 }" />
    <Scene>
      <PointLight :position="{ y: 50, z: 50 }" />
      <Box>
        <LambertMaterial />
      </Box>
    </Scene>
  </Renderer>
</template>

<script>
import { Box, Camera, LambertMaterial, PointLight, Renderer, Scene } from 'troisjs'
export default {
  components: { Box, Camera, LambertMaterial, PointLight, Renderer, Scene },
  data() {
    return {
      width: window.innerWidth,
      height: window.innerHeight,
    }
  },
}
</script>
