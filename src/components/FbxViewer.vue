<template>
    <div class="fbx-viewer" ref="root">
        <canvas ref="canvas"></canvas>
    </div>

</template>

<script setup>
import { onMounted, onBeforeUnmount, ref } from 'vue'
import * as THREE from 'three'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js'
import { FBXLoader } from 'three/examples/jsm/loaders/FBXLoader.js'

const props = defineProps({
    src: { type: String, required: true },
    background: { type: String, default: 'transparent' },
    autoRotate: { type: Boolean, default: true }
})

const root = ref(null)
const canvas = ref(null)

let renderer, scene, camera, controls, animationId

function resize() {
    if (!root.value || !renderer || !camera) return
    const { clientWidth: w, clientHeight: h } = root.value
    camera.aspect = w / h
    camera.updateProjectionMatrix()
    renderer.setSize(w, h, false)
}

onMounted(() => {
    scene = new THREE.Scene()
    if (props.background !== 'transparent') {
        scene.background = new THREE.Color(props.background)
    }

    camera = new THREE.PerspectiveCamera(50, 1, 0.1, 2000)
    camera.position.set(3, 2, 6)

    renderer = new THREE.WebGLRenderer({ canvas: canvas.value, antialias: true, alpha: props.background === 'transparent' })
    renderer.outputColorSpace = THREE.SRGBColorSpace
    renderer.shadowMap.enabled = true

    // Lights
    const hemi = new THREE.HemisphereLight(0xffffff, 0x444444, 1.0)
    hemi.position.set(0, 1, 0)
    scene.add(hemi)
    const dir = new THREE.DirectionalLight(0xffffff, 0.9)
    dir.position.set(5, 10, 7)
    dir.castShadow = true
    scene.add(dir)

    // Ground (subtle)
    const ground = new THREE.Mesh(new THREE.CircleGeometry(20, 64), new THREE.ShadowMaterial({ opacity: 0.18 }))
    ground.rotation.x = -Math.PI / 2
    ground.receiveShadow = true
    scene.add(ground)

    controls = new OrbitControls(camera, canvas.value)
    controls.enableDamping = true
    controls.autoRotate = props.autoRotate
    controls.autoRotateSpeed = 1.2

    const loader = new FBXLoader()
    loader.load(
        props.src,
        (obj) => {
            obj.traverse((c) => {
                if (c.isMesh) {
                    c.castShadow = true
                    c.receiveShadow = true
                    if (c.material) {
                        c.material.side = THREE.FrontSide
                    }
                }
            })
            // Center and scale to fit
            const box = new THREE.Box3().setFromObject(obj)
            const size = new THREE.Vector3()
            const center = new THREE.Vector3()
            box.getSize(size)
            box.getCenter(center)
            obj.position.sub(center)
            const fitHeightDistance = size.y > 0 ? size.y : 1
            const scale = 2.4 / fitHeightDistance
            obj.scale.setScalar(scale)
            scene.add(obj)
            animate()
        },
        undefined,
        (err) => {
            console.error('FBX load error', err)
        }
    )

    window.addEventListener('resize', resize)
    resize()
})

onBeforeUnmount(() => {
    window.cancelAnimationFrame(animationId)
    window.removeEventListener('resize', resize)
    controls?.dispose()
    renderer?.dispose()
})

function animate() {
    animationId = window.requestAnimationFrame(animate)
    controls.update()
    renderer.render(scene, camera)
}
</script>

<style scoped>
.fbx-viewer {
    width: 100%;
    height: 60vh;
    border-radius: 16px;
    overflow: hidden;
    background: transparent;
}

canvas {
    width: 100%;
    height: 100%;
    display: block;
}
</style>
