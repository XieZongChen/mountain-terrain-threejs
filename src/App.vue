<script setup lang="ts">
import * as THREE from 'three';
import { OrbitControls } from 'three/examples/jsm/Addons.js';
import { onMounted, ref, useTemplateRef } from 'vue';

const contentRef = useTemplateRef('content');
onMounted(() => {
  initThree();
  createMesh();
  animate();
});

let renderer: THREE.WebGLRenderer | null = null;
let scene: THREE.Scene | null = null;
let camera: THREE.PerspectiveCamera | null = null;

const initThree = () => {
  if (!contentRef.value) return;
  // 初始化场景
  scene = new THREE.Scene();

  // 初始化坐标系
  const axesHelper = new THREE.AxesHelper(200);
  scene.add(axesHelper);

  const width = window.innerWidth;
  const height = window.innerHeight;

  // 初始化相机
  camera = new THREE.PerspectiveCamera(60, width / height, 1, 10000);
  camera.position.set(200, 200, 200);
  camera.lookAt(0, 0, 0);

  // 初始化渲染器
  renderer = new THREE.WebGLRenderer({ canvas: contentRef.value });
  renderer.setSize(width, height);

  const controls = new OrbitControls(camera, renderer.domElement);
};

const createMesh = () => {
  if (!scene) return;

  // 创建一个 300 * 300 的平面几何体，宽高分成 10 段
  const geometry = new THREE.PlaneGeometry(300, 300, 10, 10);

  const positions = geometry.attributes.position;

  for (let i = 0; i < positions.count; i++) {
    // 随机顶点坐标
    positions.setZ(i, Math.random() * 50);
  }

  const material = new THREE.MeshBasicMaterial({
    color: new THREE.Color('orange'),
    wireframe: true,
  });

  const mesh = new THREE.Mesh(geometry, material);
  mesh.rotateX(Math.PI / 2); // 绕 x 轴旋转 90 度
  scene.add(mesh);
};

const animate = () => {
  if (!renderer || !scene || !camera) return;
  requestAnimationFrame(animate);

  renderer.render(scene, camera);
};
</script>

<template>
  <canvas ref="content"></canvas>
</template>

<style scoped></style>
