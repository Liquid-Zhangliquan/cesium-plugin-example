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
    async initViewer() {
      debugger;
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
        // terrainProvider: Cesium.createWorldTerrain(),
        // imageryProvider: imageryProvider,
        contextOptions: {
          webgl: {
            stencil: true,
            preserveDrawingBuffer: true,
          },
        },
      });
      window.viewer = viewer;
      viewer.scene.terrainProvider = await Cesium.ArcGISTiledElevationTerrainProvider.fromUrl(
        'https://elevation3d.arcgis.com/arcgis/rest/services/WorldElevation3D/Terrain3D/ImageServer'
      );
      viewer._cesiumWidget._creditContainer.style.display = 'none'; // 隐藏版权
      viewer.scene.postProcessStages.fxaa.enabled = true;
      // viewer.scene.globe.enableLighting = true;
      viewer.scene.globe.depthTestAgainstTerrain = true;
      this.initLayer();
    },
    async initLayer() {
      let mesh = {
        data: null,
        primitives: null,
      };
      const res = await this.getSource(`/datas/wind_zj.json`);
      // mesh.data = this.computeTemperature(res);
      // this.createGroundPolygon(mesh);
      // viewer.scene.primitives.add(mesh.primitives);
      const instances = this._getMergeInstances(res);
      const primitive = this._createPrimitive(instances);
      viewer.scene.primitives.add(primitive);
      // viewer.flyTo(primitive);
    },
    getSource(url) {
      return new Promise((resolve, reject) => {
        fetch(url)
          .then((b) => {
            return b.json();
          })
          .then((val) => {
            resolve(val);
          })
          .catch((err) => {
            reject(err);
          });
      });
    },
    _createGeometry(rings) {
      const polygon = new Cesium.PolygonGeometry({
        polygonHierarchy: new Cesium.PolygonHierarchy(Cesium.Cartesian3.fromDegreesArray(rings.toString().split(','))),
        vertexFormat: Cesium.PerInstanceColorAppearance.VERTEX_FORMAT,
      });
      // const geometry = Cesium.PolygonGeometry.createGeometry(polygon);
      return polygon;
    },
    _createGeometryInstance(geometry, properties) {
      const instance = new Cesium.GeometryInstance({
        geometry: geometry,
        attributes: {
          color: Cesium.ColorGeometryInstanceAttribute.fromColor(
            Cesium.Color.fromCssColorString(properties.color).withAlpha(0.6)
          ),
        },
      });
      return instance;
    },

    _getMergeInstances(res) {
      console.time('getInstances');
      let instances = [];
      res.features.map((feature) => {
        const { geometry, properties } = feature;
        geometry.coordinates.map((rings) => {
          const geometry = this._createGeometry(rings);
          const instance = this._createGeometryInstance(geometry, properties);
          instances.push(instance);
        });
      });
      console.timeEnd('getInstances');
      return instances;
    },

    _createPrimitive(instances) {
      const Primitive = new Cesium.GroundPrimitive({
        geometryInstances: instances,
        appearance: new Cesium.PerInstanceColorAppearance({
          // 为每个instance着色
          translucent: true,
          closed: false,
        }),
        asynchronous: false, // 确定基元是异步创建还是阻塞直到准备就绪
      });
      return Primitive;
    },
    computeTemperature(data) {
      let result = {};
      let index = 0;
      data.features.forEach((element) => {
        result[element.properties.color] = {
          result: [],
          color: element.properties.color,
          contour_value: element.properties.contour_value,
        };
        if (element.geometry.type === 'MultiPolygon') {
          element.geometry.coordinates.forEach((point) => {
            point.forEach((ele) => {
              index = 0;
              result[element.properties.color].result.push(Cesium.Cartesian3.fromDegreesArray(ele.flat()));
              index++;
            });
          });
        } else if (element.geometry.type === 'Polygon') {
          element.geometry.coordinates.forEach((point) => {
            result[element.properties.color].result.push(Cesium.Cartesian3.fromDegreesArray(point.flat()));
          });
        }
      });
      return result;
    },
    createGroundPolygon(mesh) {
      mesh['primitives'] = new Cesium.PrimitiveCollection();
      let instances = [];
      for (const key in mesh.data) {
        let color = Cesium.Color.fromCssColorString(mesh.data[key].color).withAlpha(0.4);
        mesh.data[key].result.forEach((item) => {
          const polygonHierarchy = new Cesium.PolygonHierarchy(item);
          const geometry = new Cesium.PolygonGeometry({
            polygonHierarchy,
            vertexFormat: Cesium.PerInstanceColorAppearance.VERTEX_FORMAT,
            // vertexFormat: Cesium.MaterialAppearance.MaterialSupport.ALL.vertexFormat,
          });
          instances.push(
            new Cesium.GeometryInstance({
              geometry,
              attributes: {
                color: Cesium.ColorGeometryInstanceAttribute.fromColor(color),
              },
            })
          );
        });
      }
      const primitive = new Cesium.Primitive({
        geometryInstances: instances,
        appearance: new Cesium.PerInstanceColorAppearance({
          // flat: true,
          translucent: true,
          closed: false,
        }),
      });
      mesh['primitives'].add(primitive);
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
