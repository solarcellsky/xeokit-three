<template>
  <div id="my-three"></div>
</template>
<script>
import * as THREE from "three";
import Stats from "three/addons/libs/stats.module.js";
import { OrbitControls } from "three/addons/controls/OrbitControls.js";
import { FBXLoader } from "three/addons/loaders/FBXLoader.js";

export default {
  data() {
    return {};
  },
  mounted() {
    this.initThree();
  },
  methods: {
    initThree() {
      const container = document.getElementById("my-three");
      const stats = new Stats();
      const worldScene = new THREE.Scene();
      const camera = new THREE.PerspectiveCamera(
        45,
        window.innerWidth / window.innerHeight,
        1,
        2000
      );

      camera.position.set(100, 100, 300)
      camera.lookAt(0, 0, 0)
      
      worldScene.background = new THREE.Color(0xa0a0a0);
      worldScene.fog = new THREE.Fog(0xa0a0a0, 200, 1000);

      const hemiLight = new THREE.HemisphereLight(0xffffff, 0x444444);
      hemiLight.position.set(0, 200, 0);
      worldScene.add(hemiLight);

      const dirLight = new THREE.DirectionalLight(0xffffff);
      dirLight.position.set(0, 200, 100);
      dirLight.castShadow = true;
      dirLight.shadow.camera.top = 180;
      dirLight.shadow.camera.bottom = -100;
      dirLight.shadow.camera.left = -120;
      dirLight.shadow.camera.right = 120;
      worldScene.add(dirLight);

      let cameraHelper = new THREE.CameraHelper(dirLight.shadow.camera)
      worldScene.add(cameraHelper)

      // ground
      const mesh = new THREE.Mesh(
        new THREE.PlaneGeometry(20000, 20000),
        new THREE.MeshPhongMaterial({ color: 0x999999, depthWrite: false })
      );
      mesh.rotation.x = -Math.PI / 2;
      mesh.receiveShadow = true;
      worldScene.add(mesh);

      const grid = new THREE.GridHelper(2000, 100, 0x000000, 0x000000);
      grid.material.opacity = 0.2;
      grid.material.transparent = true;
      worldScene.add(grid);

      const renderer = new THREE.WebGLRenderer({ antialias: true });
      renderer.setPixelRatio(window.devicePixelRatio);
      renderer.setSize(window.innerWidth, window.innerHeight);
      renderer.shadowMap.enabled = true;
      container.appendChild(renderer.domElement);

      

      // stats
      container.appendChild(stats.dom);

      const controls = new OrbitControls( camera, renderer.domElement);
			controls.target.set( 0, 100, 0 );
			controls.enablePan = false;
			controls.enableDamping = true;

      window.controls = controls;
      window.camera = camera;
      window.renderer = renderer;
      window.worldScene = worldScene ;
      window.stats = stats;

      this.animate();
    },
    animate() {
      requestAnimationFrame(this.animate);
      window.controls.update();
      window.camera.updateProjectionMatrix();
      window.renderer.render(window.worldScene, window.camera);
      window.stats.update();
    },
  },
};
</script>
<style lang="scss">
.about {
  height: 100%;
  width: 100%;
  background: lightblue;
  background-image: linear-gradient(lightblue, white);
}
</style>
