<script setup>
import * as THREE from "three";
import { OrbitControls } from "three/examples/jsm/Addons.js";
import * as dat from "lil-gui";
import gsap from "gsap";

const canvas = ref(null);

// function initiateThree() {

// }

onMounted(() => {
  /**
   * Debug
   */
  const gui = new dat.GUI();

  const parameters = {
    materialColor: "#ffeded",
  };

  gui.addColor(parameters, "materialColor").onChange(() => {
    material.color.set(parameters.materialColor);
    particleMaterial.color.set(parameters.materialColor);
  });

  // Set up the scene, camera, and renderer
  const scene = new THREE.Scene();

  // texture
  const textureLoader = new THREE.TextureLoader();
  const gradientTexture = textureLoader.load("/textures/gradients/3.jpg");
  gradientTexture.magFilter = THREE.NearestFilter;
  // objects
  const material = new THREE.MeshToonMaterial({
    color: parameters.materialColor,
    gradientMap: gradientTexture,
  });

  // mesh
  const objectsDistance = 4;
  const mesh1 = new THREE.Mesh(
    new THREE.TorusGeometry(1, 0.4, 16, 60),
    material
  );
  const mesh2 = new THREE.Mesh(new THREE.ConeGeometry(1, 2, 32), material);
  const mesh3 = new THREE.Mesh(
    new THREE.TorusKnotGeometry(0.8, 0.35, 100, 10),
    material
  );
  

  const verticesOfCube = [
    -1,-1,-1,    1,-1,-1,    1, 1,-1,    -1, 1,-1,
    -1,-1, 1,    1,-1, 1,    1, 1, 1,    -1, 1, 1,
];

const indicesOfFaces = [
    2,1,0,    0,3,2,
    0,4,7,    7,3,0,
    0,1,5,    5,4,0,
    1,2,6,    6,5,1,
    2,3,7,    7,6,2,
    4,5,6,    6,7,4
];

  const mesh4 = new THREE.Mesh(
    new THREE.PolyhedronGeometry( verticesOfCube, indicesOfFaces, 1, 2 ),
    material
  );

  mesh1.position.y = -objectsDistance * 0;
  mesh2.position.y = -objectsDistance * 1;
  mesh3.position.y = -objectsDistance * 2;
  mesh4.position.y = -objectsDistance * 3;

  mesh1.position.x = 2;
  mesh2.position.x = -2;
  mesh3.position.x = 2;
  mesh4.position.x = -2;
  scene.add(mesh1, mesh2, mesh3, mesh4);

  const sectionMeshes = [mesh1, mesh2, mesh3, mesh4];

  // particles
  // geometry
  const particlesCount = 200;
  const position = new Float32Array(particlesCount * 3);
  for (let i = 0; i < particlesCount; i++) {
    position[i * 3 + 0] = (Math.random() - 0.5) * 10;
    position[i * 3 + 1] =
      objectsDistance * 0.5 -
      Math.random() * objectsDistance * sectionMeshes.length;
    position[i * 3 + 2] = (Math.random() - 0.5) * 10;
  }

  const particleGeometry = new THREE.BufferGeometry();
  particleGeometry.setAttribute(
    "position",
    new THREE.BufferAttribute(position, 3)
  );

  // material
  const particleMaterial = new THREE.PointsMaterial({
    color: parameters.materialColor,
    sizeAttenuation: true,
    size: 0.03,
  });

  const particles = new THREE.Points(particleGeometry, particleMaterial);
  scene.add(particles);

  // lights
  const directionalLight = new THREE.DirectionalLight("#ffffff", 1);
  directionalLight.position.set(1, 1, 0);
  scene.add(directionalLight);

  /**
   * Sizes
   */
  const sizes = {
    width: window.innerWidth,
    height: window.innerHeight,
  };

  window.addEventListener("resize", () => {
    // Update sizes
    sizes.width = window.innerWidth;
    sizes.height = window.innerHeight;

    // Update camera
    camera.aspect = sizes.width / sizes.height;
    camera.updateProjectionMatrix();

    // Update renderer
    renderer.setSize(sizes.width, sizes.height);
    renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
  });

  /**
   * Camera
   */

  // group
  const cameraGroup = new THREE.Group();
  scene.add(cameraGroup);

  // Base camera
  const camera = new THREE.PerspectiveCamera(
    35,
    sizes.width / sizes.height,
    0.1,
    100
  );
  camera.position.z = 6;
  cameraGroup.add(camera);

  /**
   * Renderer
   */
  const renderer = new THREE.WebGLRenderer({
    canvas: canvas.value,
    alpha: true,
  });
  renderer.setSize(sizes.width, sizes.height);
  renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));

  // scroll
  let scrollY = window.scrollY;
  let currentSection = 0;

  window.addEventListener("scroll", () => {
    scrollY = window.scrollY;

    const newSection = Math.round(scrollY / sizes.height);

    if (newSection != currentSection) {
      currentSection = newSection;

      gsap.to(sectionMeshes[currentSection].rotation, {
        duration: 1.5,
        ease: "power2.inout",
        x: "+=6",
        y: "+=3",
        z: "+= 1.5",
      });
    }
  });

  // cursor
  const cursor = {};
  cursor.x = 0;
  cursor.y = 0;

  window.addEventListener("mousemove", (event) => {
    cursor.x = event.clientX / sizes.width - 0.5;
    cursor.y = event.clientY / sizes.height - 0.5;
  });

  /**
   * Animate
   */
  const clock = new THREE.Clock();
  let previousTime = 0;

  const tick = () => {
    const elapsedTime = clock.getElapsedTime();
    const deltaTime = elapsedTime - previousTime;
    previousTime = elapsedTime;

    // animate camera
    camera.position.y = (-scrollY / sizes.height) * objectsDistance;

    const parallaxX = cursor.x * 0.5;
    const parallaxY = -cursor.y * 0.5;
    cameraGroup.position.x +=
      (parallaxX - cameraGroup.position.x) * 2 * deltaTime;
    cameraGroup.position.y +=
      (parallaxY - cameraGroup.position.y) * 2 * deltaTime;

    // animate
    for (const mesh of sectionMeshes) {
      mesh.rotation.x += deltaTime * 0.1;
      mesh.rotation.y += deltaTime * 0.12;
    }
    // Render
    renderer.render(scene, camera);

    // Call tick again on the next frame
    window.requestAnimationFrame(tick);
  };

  tick();
});
</script>

<template>
  <canvas ref="canvas" class="canvas-container"></canvas>
</template>

<style scoped></style>
