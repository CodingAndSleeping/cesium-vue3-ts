<script setup lang="ts">
import * as Cesium from "cesium";

import { onMounted } from "vue";

let viewer: Cesium.Viewer;

onMounted(() => {
  init();

  addModel(
    new URL("../assets/model/glb/Cesium_Air.glb", import.meta.url).href,
    120,
    36,
    200
  );
});

// 初始化地图
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

// 加载三维模型
function addModel(
  url: string,
  lon: number,
  lat: number,
  height: number = 0,
  heading: number = 0,
  pitch: number = 0,
  roll: number = 0
): void {
  viewer.entities.removeAll();
  // 将经纬度转换为笛卡尔坐标系的三维空间坐标
  const position: Cesium.Cartesian3 = Cesium.Cartesian3.fromDegrees(
    lon,
    lat,
    height
  );
  // 设置模型的 heading、pitch、roll
  const headingpitchroll: Cesium.HeadingPitchRoll = new Cesium.HeadingPitchRoll(
    heading,
    pitch,
    roll
  );

  // 以position作为中心点将heading、pitch、roll转为四元数（ Quaternion:四元数）
  const orientation: Cesium.Quaternion =
    Cesium.Transforms.headingPitchRollQuaternion(position, headingpitchroll);

  // 向场景中添加实体
  const entity = viewer.entities.add({
    name: url,
    position: position,
    orientation: new Cesium.ConstantProperty(orientation),
    model: {
      uri: url, // 模型的路径
      minimumPixelSize: 128, // 模型最小像素大小
      maximumScale: 20000, // 模型最大缩放比例
      color: Cesium.Color.fromCssColorString("#0000FF").withAlpha(0.8), // 外表颜色
      colorBlendMode: Cesium.ColorBlendMode.REPLACE, // 可选配置选项有 HIGHLIGHT、MIX、REPLACE
      colorBlendAmount: 0.4, // 当colorBlendMode设置为MIX时，调整混合比例
      silhouetteColor: Cesium.Color.fromCssColorString("#FF0000").withAlpha(1),  // 轮廓颜色 
      silhouetteSize: 5, // 轮廓粗细
    },
  });
  // 实体跟踪视角
  viewer.trackedEntity = entity;
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
