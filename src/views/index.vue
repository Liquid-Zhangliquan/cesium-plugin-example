<template>
  <div class="base-earth-main-wrap">
    <div ref="map3d" class="map-container" id="viewerContainer"></div>
  </div>
</template>
<script>
import { defaultAccessToken } from '@/utils/utils';
export default {
  name: 'BaseMap',
  components: {},
  data() {
    return {};
  },
  beforeCreate() {},
  mounted() {
    this.initViewer();
  },
  methods: {
    initViewer() {
      Cesium.Ion.defaultAccessToken = defaultAccessToken;
      let imageryProvider = new Cesium.ArcGisMapServerImageryProvider({
        url: 'https://map.geoq.cn/arcgis/rest/services/ChinaOnlineStreetPurplishBlue/MapServer',
      });
      let viewer = new Cesium.Viewer('viewerContainer', {
        geocoder: false, // 地理位置查询定位控件
        homeButton: false, // 默认相机位置控件
        timeline: false, // 时间滚动条控件
        sceneModePicker: false,
        selectionIndicator: false, // 默认相机聚焦的选中框
        navigationHelpButton: false, // 默认的相机控制提示控件
        fullscreenButton: false, // 全屏控件
        // scene3DOnly: true, // 每个几何实例仅以3D渲染以节省GPU内存
        avigationHelpButton: false, //Whether to display the help button in the upper right corner
        navigationInstructionsInitiallyVisible: false,
        creditContainer: undefined,
        baseLayerPicker: false, // 底图切换控件
        animation: false, // 控制场景动画的播放速度控件
        shouldAnimate: true,
        infoBox: false,
        // terrainProvider: new Cesium.CesiumTerrainProvider({
        //   url: 'http://data.marsgis.cn/terrain/',
        // }),
        // imageryProvider: imageryProvider,
        contextOptions: {
          webgl: {
            stencil: true,
            preserveDrawingBuffer: true,
          },
        },
      });
      window.viewer = viewer;
      viewer._cesiumWidget._creditContainer.style.display = 'none'; // 隐藏版权
      viewer.scene.postProcessStages.fxaa.enabled = true;
      // viewer.scene.globe.enableLighting = true;
      viewer.scene.globe.depthTestAgainstTerrain = true;
    },
  },
};
</script>
<style lang="less">
.base-earth-main-wrap {
  width: 100%;
  height: 100%;
  .map-container {
    width: 100%;
    height: 100%;
  }
}
</style>
