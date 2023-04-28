<script lang="ts" setup>
import * as Cesium from "cesium";

import { onMounted, reactive, ref } from "vue";

let viewer: Cesium.Viewer;

let infoBox = ref<HTMLDivElement>();

let selectInfo = reactive<Record<string, any>>({});

onMounted(() => {
  init();
  addThreeDTilesModel();
  featurePick();
});

function init(): void {
  viewer = new Cesium.Viewer("cesiumContainer", {
    imageryProvider: new Cesium.ArcGisMapServerImageryProvider({
      // 影像服务
      url: "https://services.arcgisonline.com/ArcGIS/rest/services/World_Imagery/MapServer",
    }),

    sceneMode: Cesium.SceneMode.SCENE3D, // 显示模式
    animation: false, // 动画控件
    baseLayerPicker: false, // 图层选择器
    geocoder: false, // 搜索框
    timeline: false, // 时间线
    fullscreenButton: false, // 全屏按钮
    homeButton: false, // 主页按钮
    creditContainer: document.createElement("div"), // 指定版权信息放在某个dom元素中
    sceneModePicker: false, //显示模式切换按钮
    navigationHelpButton: false, // 帮助按钮
    vrButton: false, //  vr模式按钮
    infoBox: false, // 是否展示信息框（这个我并没有发现信息框在哪里）
  });
}

// 加载3DTiles数据
async function addThreeDTilesModel(): Promise<void> {
  // 开启地形深度检测
  viewer.scene.globe.depthTestAgainstTerrain = true;

  // 引入3Dtileset模型
  let tileset = await new Cesium.Cesium3DTileset({
    url: new URL(
      "../assets/model/Tileset/buildexample/tileset.json",
      import.meta.url
    ).href,
  }).readyPromise;

  // 往场景的primitives上添加tileset
  viewer.scene.primitives.add(tileset);

  // 相机移动到3Dtiles实体
  viewer.zoomTo(
    tileset,
    new Cesium.HeadingPitchRange(0.0, -0.5, tileset.boundingSphere.radius * 2.0)
  );
}

interface Iselected {
  feature: any;
  color: Cesium.Color;
}
function featurePick() {
  const initColor: Cesium.Color = new Cesium.Color();

  const selected: Iselected = {
    feature: undefined,
    color: Cesium.Color.BLUE,
  };

  // 获取点击时的函数
  const clickHandler = viewer.screenSpaceEventHandler.getInputAction(
    Cesium.ScreenSpaceEventType.LEFT_CLICK
  );

  viewer.screenSpaceEventHandler.setInputAction(
    (e: Cesium.ScreenSpaceEventHandler.PositionedEvent) => {
      const pickedFeature: Cesium.Cesium3DTileFeature = viewer.scene.pick(
        e.position
      );

      if (selected.feature === pickedFeature) return;

      if (
        Cesium.defined(selected.feature) &&
        selected.feature !== pickedFeature
      ) {
        selected.feature.color = initColor;
        selected.feature = undefined;
        infoBox.value!.style.display = "none";
      }

      if (!Cesium.defined(pickedFeature)) return;

      selected.feature = pickedFeature;

      selected.feature.color = selected.color;

      const ids = pickedFeature.getPropertyIds();
      for (const i of ids) {
        selectInfo[i] = pickedFeature.getProperty(i);
      }

      infoBox.value!.style.display = "block";

      infoBox.value!.style.bottom = `${window.innerHeight - e.position.y}px`;

      infoBox.value!.style.left = `${e.position.x}px`;
    },
    Cesium.ScreenSpaceEventType.LEFT_CLICK
  );
}
</script>

<template>
  <div class="container">
    <div id="cesiumContainer"></div>
  </div>

  <div ref="infoBox" class="infoBox">
    <div v-for="(item, index) in selectInfo">
      <span>{{ index }}:</span>
      <span>{{ item }}</span>
    </div>
  </div>
</template>

<style scoped lang="scss">
.container {
  #cesiumContainer {
    height: 100%;
    width: 100%;
  }
}

.infoBox {
  display: none;
  position: absolute;
  bottom: 0;
  left: 0;
  pointer-events: 0;
  border: 1px solid red;
}
</style>
