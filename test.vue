<template>
  <div class="three_globe g-wrap">
    <div class="g-hp100 g-wp100" ref="threejs-globe"></div>
  </div>
</template>
<script>
import { getCurrentInstance, onMounted } from 'vue'
export default {
  name: 'three_globe',
  expose: [],
  components: {},
  setup() {
    const { proxy } = getCurrentInstance()

    let canvas, canvas_width, canvas_height

    let mesh, renderer, scene, camera, controls

    let lightProbe
    let directionalLight

    // linear color space
    const API = {
      lightProbeIntensity: 1.0,
      directionalLightIntensity: 0.2,
      envMapIntensity: 5
    }

    const getCanvasReat = () => {
      return new Promise((resolve) => {
        proxy.$nextTick(() => {
          canvas = proxy.$refs['threejs-globe']
          var reat = proxy.getClientRect(canvas)
          canvas_width = reat.width
          canvas_height = reat.height
          resolve(reat)
        })
      })
    }

    const initScene = () => {
      scene = new proxy.$THREE.Scene()
    }

    const initThree = () => {
      return new Promise((resolve) => {
        renderer = new proxy.$THREE.WebGLRenderer( { antialias: true, alpha: true } )
        renderer.setPixelRatio(window.devicePixelRatio)
        renderer.setSize(canvas_width, canvas_height)
        canvas.appendChild(renderer.domElement)

        // tone mapping
        renderer.toneMapping = proxy.$THREE.NoToneMapping
        renderer.outputEncoding = proxy.$THREE.sRGBEncoding
        resolve()
      })
    }

    const initCamera = () => {
      return new Promise((resolve) => {
        camera = new proxy.$THREE.PerspectiveCamera(18, canvas_width / canvas_height, 1, 1000)
        camera.position.set(5, 5, 30)
        resolve()
      })
    }

    const initControls = () => {
      return new Promise((resolve) => {
        controls = new proxy.$OrbitControls(camera, renderer.domElement)
        controls.addEventListener('change', render)
        controls.minDistance = 10
        controls.maxDistance = 50
        controls.enablePan = false
        controls.autoRotate = true
        controls.autoRotateSpeed = 5
        resolve()
      })
    }

    const initLight = () => {
      return new Promise((resolve) => {
        // probe
        lightProbe = new proxy.$THREE.LightProbe()
        scene.add(lightProbe)

        // light
        directionalLight = new proxy.$THREE.DirectionalLight(0xffffff, API.directionalLightIntensity)
        directionalLight.position.set(20, 15, 150)
        scene.add(directionalLight)
        resolve()
      })
    }

    const render = () => {
      renderer.render(scene, camera)
    }

    const initEnvmap = () => {
      return new Promise((resolve) => {
        // // envmap
        // ????????????????????????loader
        // const textureLoader = new proxy.$THREE.TextureLoader()
        // // ????????????????????????
        // textureLoader.load(require('@/assets/img/login/globe.jpg'), function(texture) {
        //   // ??????????????????????????????????????????????????????mesh???????????????????????????????????????????????????????????????????????????
        //   const material = new proxy.$THREE.MeshLambertMaterial({
        //     map: texture // ????????????map??????????????????????????????
        //   })
        //   const geometry = new proxy.$THREE.SphereGeometry(5, 32, 32) // ??????????????????

        //   mesh = new proxy.$THREE.Mesh(geometry, material) // ????????????mesh??????

        //   scene.add(mesh)
        //   render()
        // })
        new proxy.$THREE.CubeTextureLoader().load([`${proxy.__static}/img/login/1.jpg`, `${proxy.__static}/img/login/2.jpg`, `${proxy.__static}/img/login/3.jpg`, `${proxy.__static}/img/login/4.jpg`, `${proxy.__static}/img/login/5.jpg`, `${proxy.__static}/img/login/6.jpg`], function(cubeTexture) {
          cubeTexture.encoding = proxy.$THREE.sRGBEncoding

          scene.background = cubeTexture

          lightProbe.copy(proxy.$LightProbeGenerator.fromCubeTexture(cubeTexture))

          const geometry = new proxy.$THREE.SphereGeometry(5, 32, 32)
          // const geometry = new proxy.$THREE.TorusKnotGeometry( 2, 64, 32, 3, 2, 3 )

          const material = new proxy.$THREE.MeshStandardMaterial({
            color: 0x000000,
            metalness: 0,
            roughness: 0,
            envMap: cubeTexture,
            envMapIntensity: API.envMapIntensity,
          })

          // mesh
          mesh = new proxy.$THREE.Mesh(geometry, material)
          scene.add(mesh)
          render()
        })
        resolve()
      })
    }

    const onWindowResize = () => {
      renderer.setSize(canvas_width, canvas_height)
      camera.aspect = canvas_width / canvas_height
      camera.updateProjectionMatrix()
      render()
    }

    const animate = () => {
      requestAnimationFrame(animate)
      if (mesh) {
        scene.rotateX(0.005)
        scene.rotateY(0.005)
        controls.update()
        render()
      }
    }

    const threeStart = async () => {
      await getCanvasReat()
      await initScene()
      await initThree()
      await initCamera()
      await initControls()
      await initLight()
      await initEnvmap()
      window.addEventListener('resize', onWindowResize)
      animate()
    }
    onMounted(() => {
      threeStart()
    })

    return {

    }
  }
}
</script>
<style lang="less" scoped>
@media screen and (max-width: 1024px) {}
</style>