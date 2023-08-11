<template>
  <canvas ref="webGl" class="webGl" />
  <div class="bottom-right" @click="throwDice">THROW</div>
</template>

<script setup lang="ts">
import { useWindowSize } from "@vueuse/core";
import { sample } from "matija-utils";
import {
  Color,
  DirectionalLight,
  IcosahedronGeometry,
  Mesh,
  MeshLambertMaterial,
  PerspectiveCamera,
  Scene,
  WebGLRenderer
} from "three";
import { OrbitControls } from "three/addons/controls/OrbitControls.js";
import { computed, onMounted, ref, watch } from "vue";

const webGl = ref();
const throwing = ref(false);

const { width, height } = useWindowSize();
const aspectRatio = computed(() => width.value / height.value);
let camera: PerspectiveCamera;
let renderer: WebGLRenderer;
let scene: Scene;
let mesh: Mesh;
let controls: OrbitControls;
let mainLight: DirectionalLight;
let interval: number;

const colors = ["orange", "pink", "purple", "red"];
const randColor = sample(colors);

const setCanvas = () => {
  scene = new Scene();
  scene.background = new Color("black");
  mainLight = new DirectionalLight("white", 2);

  const geometry = new IcosahedronGeometry();

  mesh = new Mesh(
    geometry,
    new MeshLambertMaterial({
      color: new Color(randColor)
    })
  );
  scene.add(mesh);

  camera = new PerspectiveCamera(45, aspectRatio.value, 0.1, 100);
  camera.position.z = 10;
  camera.add(mainLight);
  scene.add(camera);

  const canvas = webGl.value;
  renderer = new WebGLRenderer({ canvas, antialias: true });
  renderer.setSize(width.value, height.value);
  renderer.render(scene, camera);

  controls = new OrbitControls(camera, canvas);
  controls.enablePan = false;
  controls.enableDamping = false;
  controls.enableZoom = false;
};

const updateCamera = () => {
  camera.aspect = aspectRatio.value;
  camera.updateProjectionMatrix();
};

const updateRenderer = () => {
  renderer.setSize(width.value, height.value);
  renderer.render(scene, camera);
  renderer.setPixelRatio(window.devicePixelRatio);
};

const animate = () => {
  controls.update();
  renderer.render(scene, camera);
  requestAnimationFrame(animate);
};

watch(aspectRatio, (val) => {
  if (val) {
    updateCamera();
    updateRenderer();
  }
});

const throwDice = () => {
  if (!throwing.value) {
    throwing.value = true;
    interval = setInterval(() => {
      const time = Date.now() * 0.01;
      mesh.rotation.y = time;
      mesh.rotation.z = 0.5 * (1 + Math.sin(time));
    }, 1);
    setTimeout(() => {
      clearInterval(interval);
      throwing.value = false;
    }, 2000);
  }
};

onMounted(() => {
  setCanvas();
  animate();
});
</script>

<style scoped>
.bottom-right {
  position: fixed;
  bottom: 25px;
  right: 25px;
  color: white;
  background-color: v-bind(randColor);
  width: 55px;
  height: 55px;
  border-radius: 100%;
  font-size: 10px;
  user-select: none;
  cursor: pointer;
  font-weight: bold;
  display: flex;
  align-items: center;
  justify-content: center;
}
</style>
