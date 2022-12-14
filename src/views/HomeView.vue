<template>
  <div id="my-three">
    <div class="tree-view">
      <ul>
        <li
          v-for="(item, index) in treeViewDatas"
          :key="index"
          @mouseenter="highlightObject('on', item.uuid)"
          @mouseout="highlightObject('off', item.uuid)"
        >
          {{ item.name }}
        </li>
      </ul>
    </div>
  </div>
</template>
<script>
/* eslint-disable */
import * as THREE from "three";
import Stats from "three/addons/libs/stats.module.js";
import { OrbitControls } from "three/addons/controls/OrbitControls.js";
import { FBXLoader } from "three/addons/loaders/FBXLoader.js";
const clock = new THREE.Clock();
const stats = new Stats();

const renderer = new THREE.WebGLRenderer({ antialias: true });
renderer.setPixelRatio(window.devicePixelRatio);
renderer.setSize(window.innerWidth, window.innerHeight);
renderer.shadowMap.enabled = true;

const scene = new THREE.Scene();
const camera = new THREE.PerspectiveCamera(
  45,
  window.innerWidth / window.innerHeight,
  1,
  2000
);

const controls = new OrbitControls(camera, renderer.domElement);
controls.target.set(18.100118114326587, 56.61773169136505, -7.068393411909573);
controls.update();
controls.enablePan = true;
controls.enableDamping = true;

controls.addEventListener("change", () => {
	const $scenePerspective = {
    camera: [camera.position.x, camera.position.y, camera.position.z],
    controls: [controls.target.x, controls.target.y, controls.target.z],
  }
  console.log("scenePerspective", $scenePerspective)
})

const MAT_BUILDING_TEXTURE = new THREE.MeshPhongMaterial({
  color: new THREE.Color(0x00ff00),
  opacity: 0.6,
  transparent: true,
});
const MAT_BUILDING_OPACITY_TEXTURE = new THREE.MeshPhongMaterial({
  color: new THREE.Color(0x666666),
  opacity: 0.3,
  transparent: true,
});

export default {
  data() {
    return {
      treeViewDatas: [],
      lastObject: null,
      lastMaterial: null,
    };
  },
  mounted() {
    this.init();
  },
  methods: {
    init() {
      const self = this;
      const container = document.getElementById("my-three");
      camera.position.set(65.44148460427822, 99.28373668211103, 171.97284552703164);
      camera.lookAt(0, 0, 0);

      scene.background = new THREE.Color(0xadd8e6);
      scene.fog = new THREE.Fog(0xadd8e6, 200, 1000);

      const hemiLight = new THREE.HemisphereLight(0xffffff, 0x898989);
      hemiLight.position.set(0, 200, 0);
      scene.add(hemiLight);

      const dirLight = new THREE.DirectionalLight(0xffffff);
      dirLight.position.set(0, 200, 100);
      dirLight.castShadow = true;
      dirLight.shadow.camera.top = 180;
      dirLight.shadow.camera.bottom = -100;
      dirLight.shadow.camera.left = -120;
      dirLight.shadow.camera.right = 120;
      scene.add(dirLight);

      let cameraHelper = new THREE.CameraHelper(dirLight.shadow.camera);
      scene.add(cameraHelper);

      // ground
      const mesh = new THREE.Mesh(
        new THREE.PlaneGeometry(20000, 20000),
        new THREE.MeshPhongMaterial({ color: 0xcccccc, depthWrite: false })
      );
      mesh.rotation.x = -Math.PI / 2;
      mesh.receiveShadow = true;
      scene.add(mesh);

      const grid = new THREE.GridHelper(2000, 150, 0xcc0000, 0x409eff);
      grid.material.opacity = 0.2;
      grid.material.transparent = true;
      scene.add(grid);

      container.appendChild(renderer.domElement);

      // model
      const loader = new FBXLoader();
      const tempTreeData = [];
      loader.load("/assets/models/dt2.fbx", function (object) {
        object.scale.set(0.01, 0.01, 0.01);
        object.traverse(function (child) {
          const item = {
            uuid: child.uuid,
            name: child.name,
          };
          if (child.name) {
            tempTreeData.push(item);
          }
          if (child.isMesh) {
            child.castShadow = true;
            child.receiveShadow = true;
          }
        });

        scene.add(object);

        self.treeViewDatas = tempTreeData;
      });

      window.addEventListener("resize", this.onWindowResize());

      // stats
      container.appendChild(stats.dom);

      this.animate();
    },
    highlightObject(action, uuid) {
      const self = this;
      const actions = {
        on: () => {
          scene.traverse((child) => {
            if (child.uuid == uuid) {
              self.lastObject = child;
              self.lastMaterial = child.material;
              child.material = MAT_BUILDING_TEXTURE;
            }
          });
        },
        off: () => {
          scene.traverse((child) => {
            if (child.uuid == self.lastObject.uuid) {
              child.material = self.lastMaterial;
            }
          });
        },
      };
      actions[action]();
    },
    animate() {
      requestAnimationFrame(this.animate);
      controls.update();
      camera.updateProjectionMatrix();
      renderer.render(scene, camera);
      stats.update();
    },
    onWindowResize() {
      camera.aspect = window.innerWidth / window.innerHeight;
      camera.updateProjectionMatrix();
      renderer.setSize(window.innerWidth, window.innerHeight);
    },
  },
};
</script>
<style lang="scss">
.about {
  height: 100%;
  width: 100%;
  background: lightblue;
  background-image: linear-gradient(#add8e6, #ffffff);
}
.tree-view {
  height: 100%;
  width: 360px;
  position: absolute;
  right: 0;
  top: 0;
  background: #fefefe;
  padding: 20px;
  box-shadow: 0 0 10px rgba($color: #000000, $alpha: 0.2);
  & > ul {
    padding: 0;
    margin: 0;
    text-align: left;
    height: 100%;
    overflow: auto;
    & > li {
      font-size: 12px;
      border-bottom: 1px solid #EBEEF5;
      padding: 5px 0;
      cursor: pointer;
      &:hover {
        color: #C00;
      }
    }
  }
}
</style>
