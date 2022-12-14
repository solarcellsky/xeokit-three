<template>
  <div class="about">
    <div id="time">Loading JavaScript modules...</div>
    <canvas id="myCanvas"></canvas>
    <canvas id="myNavCubeCanvas"></canvas>
    <div id="treeViewContainer">
      <h3>Entities</h3>
      <div id="treeViewWrapper" class="treeViewWrapper"></div>
    </div>
    <div id="annotationsContainer">
      <h3>Annotations</h3>
      <div class="annotations-wrapper">
        <div
          class="annotations-item"
          v-for="(item, index) in annotationDatas"
          :key="index"
          @mouseenter="showAnnotation(item)"
          @mouseleave="hideAnnotation(item)"
        >
          <span :style="`background:${item.values.markerBGColor}`">{{ item.values.glyph }}</span>
          <h4>{{ item.values.title }}</h4>
          <p>{{ item.values.description }}</p>
        </div>
      </div>
    </div>
    <div ref="infoBox" class="info-box" v-if="isShow">
      <div><input placeholder="name" /></div>
      <div><input placeholder="description" /></div>
      <div><input placeholder="marker color" /></div>
      <div>
        <button @click="cancel">CANCEL</button>
        <button @click="submit">SUBMIT</button>
      </div>
    </div>
  </div>
</template>
<script>
import {
  Viewer,
  // WebIFCLoaderPlugin,
  LocaleService,
  XKTLoaderPlugin,
  NavCubePlugin,
  TreeViewPlugin,
  AnnotationsPlugin,
  math,
} from "@xeokit/xeokit-sdk";

export default {
  data() {
    return {
      viewer: null,
      annotations: null,
      isShow: false,
      annotationDatas: [
        {
          id: "myAnnotation1",
          objectId: "0_nanN5VH5eARTiW6lUQd3",
          worldPos: [1842012.9573374535, 12.470100610210098, -5173301.570220059],
          occludable: true,
          markerShown: true,
          labelShown: false,
          values: {
            glyph: "A1",
            title: "小窗",
            description: "正面二楼小窗",
            markerBGColor: "#FF0000",
          },
        },
        {
          id: "myAnnotation2",
          objectId: "1PAxaJRu13gfudq_tpVBm7",
          worldPos: [1842011.4075617846, 15.162103363238742, -5173298.842028536],
          occludable: true,
          markerShown: true,
          labelShown: false,
          values: {
            glyph: "A2",
            title: "小窗",
            description: "左侧二楼小窗1",
            markerBGColor: "#0000FF",
          },
        },
        {
          id: "myAnnotation3",
          objectId: "2JBh8Xn6DDFfR$iL4REVIO",
          worldPos: [1842011.073563822, 12.791727313290354, -5173294.341529159],
          occludable: true,
          markerShown: true,
          labelShown: false,
          values: {
            glyph: "A3",
            title: "小窗",
            description: "左侧三楼小窗2",
            markerBGColor: "#00FF00",
          },
        },
      ],
    };
  },
  mounted() {
    this.init();
  },
  methods: {
    init() {
      const viewer = new Viewer({
        canvasId: "myCanvas",
        transparent: true,
        localeService: new LocaleService({
          messages: {
            en: {
              // English
              NavCube: {
                front: "Front",
                back: "Back",
                top: "Top",
                bottom: "Bottom",
                left: "Left",
                right: "Right",
              },
            },
            "zh-cn": {
              // zh-cn
              NavCube: {
                front: "前",
                back: "后",
                top: "上",
                bottom: "下",
                left: "左",
                right: "右",
              },
            },
          },
          locale: "zh-cn",
        }),
      });

      viewer.localeService.locale = "zh-cn";

      this.viewer = viewer;

      //------------------------------------------------------------------------------------------------------------------
      // Create an AnnotationsPlugin, with which we'll create annotations
      //------------------------------------------------------------------------------------------------------------------

      const annotations = new AnnotationsPlugin(viewer, {
        markerHTML:
          "<div class='annotation-marker' style='background-color: {{markerBGColor}};'>{{glyph}}</div>",
        labelHTML:
          "<div class='annotation-label' style='background-color: {{labelBGColor}};'>\
            <div class='annotation-title'>{{title}}</div>\
            <div class='annotation-desc'>{{description}}</div>\
            </div>",

        values: {
          markerBGColor: "#FF0000",
          labelBGColor: "#FFFFFF",
          glyph: "M",
          title: "Untitled",
          description: "No description",
        },
      });

      this.annotations = annotations;

      this.makeScene();
    },
    makeScene() {
      const viewer = this.viewer;
      const annotations = this.annotations;
      const cameraControl = viewer.cameraControl;
      const cameraFlight = viewer.cameraFlight;
      const scene = viewer.scene;
      const camera = viewer.camera;

      const boundary = scene.canvas.boundary;
      const left = boundary[0];
      const top = boundary[1];
      const offsetX = -260;
      const offsetY = -17;

      cameraControl.doublePickFlyTo = true;
      cameraControl.followPointer = true;
      cameraFlight.duration = 1.0;
      cameraFlight.fitFOV = 25;

      // camera.eye = [-2.56, 8.38, 8.27];
      // camera.look = [13.44, 3.31, -14.83];
      // camera.up = [0.1, 0.98, -0.14];

      camera.eye = [1841982.5187600704, 19.207790938410042, -5173303.042326414];
      camera.look = [1842011.793756829, 9.913817421536704, -5173299.841616623];
      camera.up = [0.2991762376746394, 0.9536370664170352, 0.0327096983532173];

      scene.xrayMaterial.fillAlpha = 0.1;
      scene.xrayMaterial.fillColor = [0, 0, 1];
      scene.xrayMaterial.edgeAlpha = 0.4;
      scene.xrayMaterial.edgeColor = [0.2, 0, 0];

      // 高亮材质颜色
      scene.highlightMaterial.fill = false;
      scene.highlightMaterial.fillAlpha = 0.3;
      scene.highlightMaterial.edgeColor = [1, 1, 0];

      // 选中材质颜色
      scene.selectedMaterial.fill = false;
      scene.selectedMaterial.fillAlpha = 0.3;
      scene.selectedMaterial.edgeColor = [0, 1, 0];

      new NavCubePlugin(viewer, {
        canvasId: "myNavCubeCanvas",
        visible: true,
        color: "#ffffff",
      });

      new TreeViewPlugin(viewer, {
        containerElement: document.getElementById("treeViewWrapper"),
        autoExpandDepth: 3, // Initially expand the root tree node
      });

      // const ifcLoader = new WebIFCLoaderPlugin(viewer);

      // const sceneModel = ifcLoader.load({
      //   id: "myModel",
      //   src: "../assets/models/ifc/IfcOpenHouse2x3.ifc",
      //   loadMetadata: true, // Default
      //   excludeTypes: ["IfcSpace"],
      //   edges: true,
      // });

      const xktLoader = new XKTLoaderPlugin(viewer);
      const sceneModel = xktLoader.load({
        id: "myModel",
        src: "../assets/models/xkt/v7/MAP/MAP.xkt",
        metaModelSrc: "../assets/models/xkt/v7/MAP/MAP.json",
        edges: true,
        excludeTypes: ["IfcSpace"],
        objectDefaults: {}
      });

      let lastEntity = null;
      viewer.scene.input.on("mousemove", function (coords) {
        const hit = viewer.scene.pick({
          canvasPos: coords,
        });

        if (hit) {
          if (!lastEntity || hit.entity.id !== lastEntity.id) {
            if (lastEntity) {
              // lastEntity.xrayed = false;
              lastEntity.selected = false;
              // lastEntity.highlighted = false;
            }

            lastEntity = hit.entity;
            // hit.entity.xrayed = true;
            hit.entity.selected = true;
            // hit.entity.highlighted = true;
          }
        } else {
          if (lastEntity) {
            // lastEntity.xrayed = false;
            lastEntity.selected = false;
            // lastEntity.highlighted = false;
            lastEntity = null;
          }
        }
      });

      viewer.cameraControl.on("picked", (pickResult) => {
        const canvasPos = pickResult.canvasPos;
        const entity = pickResult.entity;
        const aabb = entity.aabb;
        const entityCenter = math.getAABB3Center(aabb);
        console.log("entityCenter: ", entityCenter);
        const metaObject = viewer.metaScene.metaObjects[entity.id];
        this.isShow = true;

        if (metaObject.isMarked) {
          alert("Object has marked!");
        } else {
          this.createAnnotation(annotations, {
            id: `sajndskjdkadkm${Math.random(10)}`,
            /////////////////////////////////////////  Entity
            worldPos: entityCenter,
            occludable: true,
            markerShown: true,
            labelShown: false,
            labelHTML:
              "<div class='annotation-label' style='background-color: {{labelBGColor}};'>\
                  <div class='annotation-title'>{{title}}</div>\
                  <div class='annotation-desc'>{{description}}</div>\
                  </div>",

            values: {
              glyph: "A",
              title: metaObject.name,
              description: metaObject.id,
              markerBGColor: "red",
            },
          });

          setTimeout(() => {
            const infoBox = this.$refs["infoBox"];
            infoBox.style.left =
              20 + Math.floor(left + canvasPos[0] + offsetX) + "px";
            infoBox.style.top = Math.floor(top + canvasPos[1] + offsetY) + "px";
          }, 100);

          metaObject.isMarked = true;
        }
      });

      annotations.on("markerMouseEnter", (annotation) => {
        const canvasPos = annotation.canvasPos;
        this.isShow = true;
        annotation.setLabelShown(true);
        setTimeout(() => {
          const infoBox = this.$refs["infoBox"];
          infoBox.style.left =
            20 + Math.floor(left + canvasPos[0] + offsetX) + "px";
          infoBox.style.top = Math.floor(top + canvasPos[1] + offsetY) + "px";
        }, 100);
      });

      annotations.on("markerMouseLeave", (annotation) => {
        annotation.setLabelShown(false);
      });

      const t0 = performance.now();
      document.getElementById("time").innerHTML = "Loading model...";
      sceneModel.on("loaded", function () {
        const t1 = performance.now();
        document.getElementById("time").innerHTML =
          "Model loaded in " +
          Math.floor(t1 - t0) / 1000.0 +
          " seconds. Objects: " +
          sceneModel.numEntities;
      });

      //------------------------------------------------------------------------------------------------------------------
      // Create some Annotations
      //------------------------------------------------------------------------------------------------------------------
      this.annotationDatas.map((o) => {
        this.disableMarker(o.objectId);
        this.createAnnotation(annotations, o);
      });
    },
    disableMarker(objectId) {
      let metaObject = null;
      setTimeout(() => {
        metaObject = this.viewer.metaScene.metaObjects[objectId];
        if (metaObject) metaObject.isMarked = true;
      }, 500);
    },
    showAnnotation(target) {
      this.isShow = true;
      const scene = this.viewer.scene;
      const boundary = scene.canvas.boundary;
      const left = boundary[0];
      const top = boundary[1];
      const offsetX = -260;
      const offsetY = -17;

      const annotation = scene.components[target.id];
      const canvasPos = annotation.canvasPos;
      annotation.setLabelShown(true);

      setTimeout(() => {
        const infoBox = this.$refs["infoBox"];
        infoBox.style.left =
          20 + Math.floor(left + canvasPos[0] + offsetX) + "px";
        infoBox.style.top = Math.floor(top + canvasPos[1] + offsetY) + "px";
      }, 0);
      scene.setObjectsColorized(target.objectId, [1, 0, 1]);
    },
    hideAnnotation(target) {
      const scene = this.viewer.scene;
      const annotation = scene.components[target.id];
      annotation.setLabelShown(false);
      scene.setObjectsColorized(target.objectId, false);
      this.isShow = false;
    },
    createAnnotation(target, options) {
      target.createAnnotation(options);
    },
    cancel() {
      this.isShow = false;
    },
    submit() {
      alert("保存标记");
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
.info-box {
  position: absolute;
  z-index: 200001;
  padding: 20px;
  background: #ffffff;
  border-radius: 4px;
  box-shadow: 5px 5px 15px 1px rgba(0, 0, 0, 0.2);
  &:after {
    content: "";
    position: absolute;
    border-style: solid;
    border-width: 5px 0 5px 5px;
    border-color: transparent #ffffff;
    display: block;
    width: 0;
    z-index: 1;
    margin-top: -11px;
    right: -2px;
    top: 20px;
  }
  &:before {
    content: "";
    position: absolute;
    border-style: solid;
    border-width: 6px 0 6px 6px;
    border-color: transparent #ffffff;
    display: block;
    width: 0;
    z-index: 0;
    margin-top: -12px;
    right: -6px;
    top: 20px;
  }
}
#time {
  position: absolute;
  left: 0;
  top: 0;
  z-index: 20000;
  background: rgba(0, 0, 0, 0.5);
  color: #ffffff;
  padding: 1px 2px;
  font-size: 12px;
  font-weight: 700;
}
#myCanvas {
  width: calc(100% - 260px);
  height: 100%;
  position: absolute;
  left: 0;
  top: 0;
  z-index: 19999;
}
/* ----------------------------------------------------------------------------------------------------------*/
/* NavCubePlugin */
/* ----------------------------------------------------------------------------------------------------------*/

#myNavCubeCanvas {
  position: absolute;
  width: 150px;
  height: 150px;
  bottom: 0;
  right: 270px;
  z-index: 20000;
}

#annotationsContainer {
  pointer-events: all;
  height: 40%;
  overflow: hidden;
  position: absolute;
  background-color: rgba(255, 255, 255, 0.2);
  color: black;
  bottom: 0;
  right: 0;
  z-index: 20000;
  font-size: 12px;
  user-select: none;
  -ms-user-select: none;
  -moz-user-select: none;
  -webkit-user-select: none;
  width: 260px;
  background: #eee;
  box-shadow: 5px 5px 15px 1px rgba(0, 0, 0, 0.2);
  & h3 {
    text-align: left;
    padding: 15px 20px 5px;
    margin: 0;
    font-size: 16px;
  }
  & .annotations {
    &-wrapper {
      height: calc(100% - 38.5px);
      overflow: auto;
      padding: 0 20px;
    }
    &-item {
      display: flex;
      text-align: left;
      border-bottom: 1px solid #dcdfe6;
      cursor: pointer;
      align-items: center;
      & span {
        display: flex;
        color: #ffffff;
        width: 16px;
        height: 16px;
        font-size: 10px;
        border-radius: 4px;
        align-items: center;
        justify-content: center;
        line-height: 1;
      }
      & h4 {
        padding: 5px 0 5px 5px;
        margin: 0;
      }
      & p {
        padding: 5px;
        margin: 0;
        color: #909399;
      }
      &:hover {
        color: #c00;
      }
    }
  }
}

/* ----------------------------------------------------------------------------------------------------------*/
/* TreeViewPlugin */
/* ----------------------------------------------------------------------------------------------------------*/

#treeViewContainer {
  pointer-events: all;
  height: 60%;
  overflow: hidden;
  position: absolute;
  background-color: rgba(255, 255, 255, 0.2);
  color: #000000;
  top: 0;
  right: 0;
  z-index: 20000;
  font-size: 12px;
  user-select: none;
  -ms-user-select: none;
  -moz-user-select: none;
  -webkit-user-select: none;
  width: 260px;
  background: #eee;
  box-shadow: 5px 5px 15px 1px rgba(0, 0, 0, 0.2);
  & h3 {
    text-align: left;
    padding: 15px 20px 5px;
    margin: 0;
    font-size: 16px;
  }
  & .treeViewWrapper {
    height: calc(100% - 38.5px);
    padding: 0 10px;
    & ul {
      list-style: none;
      padding: 0 0 0 10px;
      pointer-events: none;

      & li {
        position: relative;
        pointer-events: none;
        padding-top: 3px;
        padding-bottom: 3px;
        vertical-align: middle;
        line-height: 1;

        & a {
          background-color: #eee;
          border-radius: 50%;
          color: #000;
          display: flex;
          height: 16px;
          width: 16px;
          left: -5px;
          position: absolute;
          text-align: center;
          text-decoration: none;
          pointer-events: all;
          &.plus {
            display: flex;
            align-items: center;
            justify-content: center;
            background-color: #aaa;
            pointer-events: all;
            line-height: 1;
            font-weight: 700;
          }
          &.minus {
            display: flex;
            align-items: center;
            justify-content: center;
            background-color: #ccc;
            pointer-events: all;
            line-height: 1;
            font-weight: 700;
          }
          &:active {
            top: 1px;
            pointer-events: all;
          }
        }
        & span {
          display: inline-block;
          width: calc(100% - 50px);
          padding-left: 4px;
          pointer-events: all;
          text-align: left;
          white-space: nowrap;
          overflow: hidden;
          text-overflow: ellipsis;
          border-bottom: 1px solid #dcdfe6;
          padding: 0 0 3px;
          margin: 0 0 -3px;
          &:hover {
            color: white;
            cursor: pointer;
            background: black;
            padding-left: 2px;
            pointer-events: all;
          }
        }
      }
    }

    & .highlighted-node {
      /* Appearance of node highlighted with TreeViewPlugin#showNode() */
      border: black solid 1px;
      background: yellow;
      color: black;
      padding-left: 1px;
      padding-right: 5px;
      pointer-events: all;
    }
  }
}
.annotation {
  &-title {
    font: normal 20px arial, serif;
    margin-bottom: 8px;
  }
  &-desc {
    font: normal 14px arial, serif;
  }
  &-marker {
    display: flex;
    color: #ffffff;
    line-height: 1.8;
    text-align: center;
    font-weight: bold;
    position: absolute;
    line-height: 1;
    width: 25px;
    height: 25px;
    border-radius: 15px;
    border: 1px solid #ffffff;
    background: black;
    visibility: hidden;
    box-shadow: 5px 5px 15px 1px rgba(0, 0, 0, 0.2);
    z-index: 0;
    align-items: center;
    justify-content: center;
  }
  &-label {
    position: absolute;
    max-width: 250px;
    min-height: 50px;
    padding: 8px;
    padding-left: 12px;
    padding-right: 12px;
    background: #ffffff;
    color: #000000;
    border-radius: 4px;
    box-shadow: 5px 5px 15px 1px rgba(0, 0, 0, 0.2);
    &:after {
      content: "";
      position: absolute;
      border-style: solid;
      border-width: 5px 5px 5px 0;
      border-color: transparent #ffffff;
      display: block;
      width: 0;
      z-index: 1;
      margin-top: -11px;
      left: -2px;
      top: 20px;
    }
    &:before {
      content: "";
      position: absolute;
      border-style: solid;
      border-width: 6px 6px 6px 0;
      border-color: transparent #ffffff;
      display: block;
      width: 0;
      z-index: 0;
      margin-top: -12px;
      left: -6px;
      top: 20px;
    }
  }
}
</style>
