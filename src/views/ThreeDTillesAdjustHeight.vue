<script setup lang="ts">
import * as Cesium from "cesium";

import { onMounted } from "vue";

let viewer: Cesium.Viewer;

onMounted(() => {
  init();

  addThreeDTilesModel();
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

// 加载3DTiles数据
async function addThreeDTilesModel(): Promise<void> {
  // 开启地形深度检测
  viewer.scene.globe.depthTestAgainstTerrain = false;

  // 引入3Dtileset模型
  let tileset = await new Cesium.Cesium3DTileset({
    url: new URL(
      "../assets/model/Tileset/buildexample/tileset.json",
      import.meta.url
    ).href,
  }).readyPromise;

  // 往场景的primitives上添加tileset
  viewer.scene.primitives.add(tileset);

  // 相机视角移动到tileset
  viewer.zoomTo(
    tileset,
    new Cesium.HeadingPitchRange(0.0, -1, tileset.boundingSphere.radius * 2.0)
  );

  // 定义一个初始化height
  let height = 0;
  // 设置定时器，每一秒height+1
  // setInterval(()=>{
  //   adjustTilesetHeight(tileset,height++)
  // },1000)

  adjustTilesetHeight(tileset);
}

// 调整模型高度方法
function adjustTilesetHeight(
  tileset: Cesium.Cesium3DTileset,
  height: number = 10
): void {
  // 获取模型的经纬度
  const cartographic = Cesium.Cartographic.fromCartesian(
    tileset.boundingSphere.center
  );

  // 得到模型所在经纬度的地面点的笛卡尔坐标
  const surface = Cesium.Cartesian3.fromRadians(
    cartographic.longitude,
    cartographic.latitude,
    0.0
  );

  // 设置模型所在经纬度的指定高度的笛卡尔坐标
  const offset = Cesium.Cartesian3.fromRadians(
    cartographic.longitude,
    cartographic.latitude,
    height
  );

  // 计算差向量
  const translation = Cesium.Cartesian3.subtract(
    offset,
    surface,
    new Cesium.Cartesian3()
  );

  // 设置模型的变换矩阵
  tileset.modelMatrix = Cesium.Matrix4.fromTranslation(translation);
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
