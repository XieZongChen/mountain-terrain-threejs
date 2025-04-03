<script setup lang="ts">
import { createNoise2D } from 'simplex-noise';
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
  // const axesHelper = new THREE.AxesHelper(200);
  // scene.add(axesHelper);

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

let geometry: THREE.PlaneGeometry;

const createMesh = () => {
  if (!scene) return;

  // 创建一个 300 * 300 的平面几何体，宽高分成 10 段
  geometry = new THREE.PlaneGeometry(3000, 3000, 100, 100);

  const material = new THREE.MeshBasicMaterial({
    color: new THREE.Color('orange'),
    wireframe: true,
  });

  const mesh = new THREE.Mesh(geometry, material);
  mesh.rotateX(Math.PI / 2); // 绕 x 轴旋转 90 度
  scene.add(mesh);
};

const updatePosition = () => {
  const positions = geometry.attributes.position;

  // 山脉需要有连续性的随机，而不是这种完全随机，所以需要使用噪声函数
  const noise2D = createNoise2D();

  for (let i = 0; i < positions.count; i++) {
    // 传入 x、y 让噪音算法算出这个位置的 z
    const x = positions.getX(i);
    const y = positions.getY(i);
    /**
     * 这个函数与 Math.random() 类似，返回 0 到 1 的数，只不过返回结果是与传入的 x、y 有关系的随机数
     * x、y 除以的数是为了让噪音函数的结果更平滑
     * 最终结果乘以 50 就是放大到 0 到 50
     */
    const z = noise2D(x / 300, y / 300) * 50;

    // 使用时间来计算正弦，得到的就是一个不断变化的 -1 到 1 的值，制作山脉起伏动画
    const sinNum = Math.sin(Date.now() * 0.002 + x * 0.05) * 10;

    positions.setZ(i, z + sinNum);
  }

  // 通知 WebGL（GPU）更新顶点位置，默认不更新顶点位置
  positions.needsUpdate = true;
};

const animate = () => {
  if (!renderer || !scene || !camera) return;
  updatePosition();
  renderer.render(scene, camera);
  requestAnimationFrame(animate);
};
</script>

<template>
  <canvas ref="content"></canvas>
</template>

<style scoped></style>
