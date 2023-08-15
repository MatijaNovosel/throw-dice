<template>
  <canvas ref="webGl" class="webGl" />
  <div v-show="resultN !== 0" class="result">
    {{ resultN }}
  </div>
  <div class="bottom-right">
    <btn :bg-color="randColor" @click="throwDice"> Throw </btn>
    <btn
      style="font-size: 24px; font-weight: bold"
      :bg-color="randColor"
      @click="changeType"
    >
      {{ sides[queueIdx] }}
    </btn>
  </div>
</template>

<script setup lang="ts">
import { useWindowSize } from "@vueuse/core";
import { randInt, sample } from "matija-utils";
import {
  BoxGeometry,
  Color,
  DirectionalLight,
  DodecahedronGeometry,
  IcosahedronGeometry,
  Mesh,
  MeshPhongMaterial,
  OctahedronGeometry,
  PerspectiveCamera,
  Scene,
  TetrahedronGeometry,
  WebGLRenderer
} from "three";
import { OrbitControls } from "three/addons/controls/OrbitControls.js";
import { computed, onMounted, ref, watch } from "vue";
import btn from "./components/btn.vue";

const webGl = ref();
const throwing = ref(false);
const queueIdx = ref(0);
const resultN = ref(0);

const { width, height } = useWindowSize();
const aspectRatio = computed(() => width.value / height.value);

let camera: PerspectiveCamera;
let renderer: WebGLRenderer;
let scene: Scene;
let mesh: Mesh;
let controls: OrbitControls;
let mainLight: DirectionalLight;
let interval: number;

const colors = ["orange", "salmon", "purple", "red"];
const geometries: any[] = [
  new IcosahedronGeometry(), // 20
  new DodecahedronGeometry(), // 12
  new OctahedronGeometry(), // 8
  new BoxGeometry(), // 6
  new TetrahedronGeometry() //4
];
const sides = [20, 12, 8, 6, 4];
const randColor = sample(colors);

const setCanvas = () => {
  scene = new Scene();
  scene.background = new Color("black");
  mainLight = new DirectionalLight("white", 2);
  mainLight.position.z = 15;

  const geometry = geometries[queueIdx.value];
  geometry.rotateX(95);
  geometry.rotateZ(45);

  mesh = new Mesh(
    geometry,
    new MeshPhongMaterial({
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

  camera.lookAt(mesh.position);
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
      const result = randInt(1, sides[queueIdx.value]);
      resultN.value = result;
    }, 2000);
  }
};

const changeType = () => {
  if (throwing.value) return;
  resultN.value = 0;
  if (queueIdx.value + 1 > geometries.length - 1) {
    queueIdx.value = 0;
  } else {
    queueIdx.value++;
  }
  mesh.geometry = geometries[queueIdx.value];
};

watch(aspectRatio, (val) => {
  if (val) {
    updateCamera();
    updateRenderer();
  }
});

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
  display: flex;
  gap: 10px;
}

.result {
  position: absolute;
  font-size: 64px;
  font-weight: bold;
  color: white;
  top: 25px;
  left: 25px;
}
</style>
