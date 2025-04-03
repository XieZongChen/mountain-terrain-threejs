<script setup lang="ts">
import * as THREE from 'three';
import { onMounted, ref, useTemplateRef } from 'vue';

const contentRef = useTemplateRef('content');
onMounted(() => {
  initThree();
  animate();
});

let renderer: THREE.WebGLRenderer | null = null;
let scene: THREE.Scene | null = null;
let camera: THREE.PerspectiveCamera | null = null;
let cube: THREE.Mesh | null = null;

const initThree = () => {
  if (!contentRef.value) return;
  // 初始化场景、相机和渲染器
  scene = new THREE.Scene();
  camera = new THREE.PerspectiveCamera(
    75,
    window.innerWidth / window.innerHeight,
    0.1,
    1000
  );
  camera.position.z = 5;

  renderer = new THREE.WebGLRenderer({ canvas: contentRef.value });
  renderer.setSize(window.innerWidth, window.innerHeight);

  // 创建一个立方体
  const geometry = new THREE.BoxGeometry();
  const material = new THREE.MeshBasicMaterial({ color: 0x00ff00 });
  cube = new THREE.Mesh(geometry, material);
  scene.add(cube);
};

const animate = () => {
  if (!renderer || !scene || !camera) return;
  requestAnimationFrame(animate);
  if (cube) {
    cube.rotation.x += 0.01;
    cube.rotation.y += 0.01;
  }
  renderer.render(scene, camera);
};
</script>

<template>
  <canvas ref="content"></canvas>
</template>

<style scoped></style>
