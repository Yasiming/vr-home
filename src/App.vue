<script setup lang="ts">
import {ref, onMounted} from 'vue'
import * as THREE from  'three'
import gsap from 'gsap'

const container = ref<HTMLDivElement | null>()

// 创建场景-相机
const scene = new THREE.Scene()
const camera = new THREE.PerspectiveCamera(
    75,
    window.innerWidth / window.innerHeight,
    0.1,
    1000
)
camera.position.set(0, 0, 0)

// 创建渲染器
const renderer = new THREE.WebGLRenderer()
renderer.setSize(window.innerWidth, window.innerHeight)
// 渲染
const render = () => {
  renderer.render(scene, camera)
  requestAnimationFrame(render)
}

window.addEventListener("resize", () => {
  renderer.setSize(window.innerWidth, window.innerHeight);
  camera.aspect = window.innerWidth / window.innerHeight;
  camera.updateProjectionMatrix();
});

onMounted(() => {
  if (container.value) {
    container.value.appendChild(renderer.domElement)
    render()

    let isMouseDown = false
    // 监听鼠标按下事件
    container.value?.addEventListener('mousedown', () => {
      isMouseDown = true
    }, false)

    container.value?.addEventListener('mouseup', () => {
      isMouseDown = false
    }, false)

    container.value?.addEventListener('mouseout', () => {
      isMouseDown =false
    })
    // 是否按下鼠标，移动鼠标
    container.value.addEventListener('mousemove', (event) => {
      if (isMouseDown) {
        camera.rotation.y += event.movementX * 0.001
        camera.rotation.x += event.movementY * 0.001
        camera.rotation.order = 'YXZ'
      }
    })

    // 创建客厅
    new Room('客厅', 0, './img/livingroom/');

    // 创建厨房
    const kitchenPosition = new THREE.Vector3(2, 0, 10)
    const kitEuler = new THREE.Euler(0, -Math.PI / 2, 0)
    new Room('厨房', 3, './img/kitchen/', kitchenPosition, kitEuler)

    // 创建精灵文字
    const kitchenText = new SpriteText("厨房", new THREE.Vector3(1.5, 0, 4))
    kitchenText.onClick(() => {
      gsap.to(camera.position, {
        duration: 1,
        x: kitchenPosition.x,
        y: kitchenPosition.y,
        z: kitchenPosition.z
      })
    })

    const kitchenBackTextPosition = new THREE.Vector3(1, 0, 6)
    new SpriteText("客厅", kitchenBackTextPosition).onClick(() => backToLivingRoom())

    // 阳台
    const balconyPosition = new THREE.Vector3(0, 0, -10)
    new Room('阳台', 8, './img/balcony/',balconyPosition)
    const balconyText = new SpriteText("阳台", new THREE.Vector3(0, 0, -4))
    balconyText.onClick(() => {
      gsap.to(camera.position, {
        duration: 1,
        x: balconyPosition.x,
        y: balconyPosition.y,
        z: balconyPosition.z
      })
    })
    new SpriteText('客厅', new THREE.Vector3(1, 0, -6)).onClick(() => backToLivingRoom())
  }
})

const backToLivingRoom = () => {
  gsap.to(camera.position, {
    duration: 1,
    x: 0,
    y: 0,
    z: 0
  })
}

class Room {
  name: string
  constructor(
      name: string,
      roomIndex: number,
      textureUrl: string,
      position = new THREE.Vector3(0, 0, 0),
      euler = new THREE.Euler(0, 0, 0)
  ) {
    this.name = name
    // 创建立方体
    const geometry = new THREE.BoxGeometry(10, 10, 10)
    geometry.scale(1, 1, -1)
    const arr = [
      `${roomIndex}_r`,
      `${roomIndex}_l`,
      `${roomIndex}_u`,
      `${roomIndex}_d`,
      `${roomIndex}_f`,
      `${roomIndex}_b`
    ]
    const boxMaterials: THREE.MeshBasicMaterial[] = []
    arr.forEach(item => {
      // 纹理加载
      const texture = new THREE.TextureLoader().load(textureUrl + item + '.jpg')
      boxMaterials.push(new THREE.MeshBasicMaterial({ map: texture }))
    })
    const cube = new THREE.Mesh(geometry, boxMaterials)
    cube.position.copy(position)
    cube.rotation.copy(euler)
    scene.add(cube)
  }
}

class SpriteText {
  sprite: THREE.Sprite
  callbacks: any[]
  constructor(text: string, positon: THREE.Vector3) {
    this.callbacks = []
    const canvas = document.createElement('canvas')
    canvas.width = 1024
    canvas.height = 1024
    const context = canvas.getContext('2d')!
    context.fillStyle = 'rgba(100, 100, 100, 0.7)'
    context.fillRect(0, 256, canvas.width, canvas.height / 2)
    context.textAlign = 'center'
    context.textBaseline = 'middle'
    context.font = 'bold 200px Arial'
    context.fillStyle = 'white'
    context.fillText(text, canvas.width / 2, canvas.height / 2)

    let texture = new THREE.CanvasTexture(canvas)
    const material = new THREE.SpriteMaterial({
      map: texture,
      transparent: true
    })
    const sprite = new THREE.Sprite(material);
    sprite.position.copy(positon)
    this.sprite = sprite
    scene.add(sprite)
    let mouse = new THREE.Vector2()
    const raycaster = new THREE.Raycaster()
    window.addEventListener('click', (event) => {
      mouse.x = (event.clientX / window.innerWidth) * 2 - 1
      mouse.y = -(event.clientY / window.innerHeight) * 2 + 1
      raycaster.setFromCamera(mouse, camera)
      let intersects = raycaster.intersectObject(sprite)
      if (intersects.length > 0) {
        this.callbacks.forEach((callback) => {
          callback()
        })
      }
    })
  }
  onClick(callback: () => void) {
    this.callbacks.push(callback)
  }
}
</script>

<template>
  <div class="container" ref="container"></div>
</template>

<style scoped>
.container {
  height: 100vh;
}
</style>
