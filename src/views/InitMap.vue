<script lang="ts" setup>
import * as Cesium from "cesium";
import { Viewer } from "cesium";

import { onMounted, ref } from "vue";

let viewer: Viewer;

onMounted(() => {
  init();
});

function init() {
  viewer = new Cesium.Viewer("cesiumContainer", {
    terrainProvider: new Cesium.CesiumTerrainProvider({
      // 地形服务
      url: Cesium.IonResource.fromAssetId(3956),
      requestVertexNormals: true,
      requestWaterMask: true,
    }),
    imageryProvider: new Cesium.ArcGisMapServerImageryProvider({
      // 影像服务
      url: "https://services.arcgisonline.com/ArcGIS/rest/services/World_Imagery/MapServer",
    }),
    skyBox: new Cesium.SkyBox({
      // 天空盒
      sources: {
        // 各个坐标轴的正负方向
        positiveX: new URL("@/assets/sky.jpg", import.meta.url).href,
        negativeX: new URL("@/assets/sky.jpg", import.meta.url).href,
        positiveY: new URL("@/assets/sky.jpg", import.meta.url).href,
        negativeY: new URL("@/assets/sky.jpg", import.meta.url).href,
        positiveZ: new URL("@/assets/sky.jpg", import.meta.url).href,
        negativeZ: new URL("@/assets/sky.jpg", import.meta.url).href,
      },
      // 是否显示天空盒
      show: true,
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
</script>

<template>
  <div class="container">
    <div id="cesiumContainer"></div>
  </div>
</template>

<style scoped lang="scss">
.container {
  #cesiumContainer {
    height: 100%;
    width: 100%;
  }
}
</style>
