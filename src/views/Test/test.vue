<!--  -->
<template>
  <div class=''>
    <ds-cesium @init="initMap"
               v-if="true"
               :animation="false"
               :orientation="$config.orientation"
               :options="$config.mapInitOpts"
               :center="$config.center"></ds-cesium>
  </div>
</template>

<script>

import dsCesium from "@3d-space/ds-cesium"
import "@3d-space/ds-cesium/dist/dsCesium.css"
import { loadMapInterfaceEvent, loadAreacodeLine } from "@/util/ds.cesium.common"
import { businessApi } from "@/api";
export default {
  name: 'Home',
  components: {
    dsCesium
  },
  props: {},
  data () {
    return {

    };
  },
  computed: {},
  watch: {},
  methods: {
    initMap (viewer) {
      debugger
      viewer.scene.terrainProvider = new Cesium.CesiumTerrainProvider({
        url: 'http://data.marsgis.cn/terrain/',
        requestWaterMask: true,
        requestVertexNormals: true
      });

      window.dsCesium.viewer.addIgyLayer4Provider({
        provider: new Cesium.WebMapServiceImageryProvider({
          url: this.$config.geoFujianUrl,
          layers: "fujian",
          parameters: {
            service: 'WMS',
            format: 'image/png',
            transparent: true
          }
        }),
        layerName: "fujian"
      })
      loadMapInterfaceEvent(viewer, this)
      loadAreacodeLine(viewer, this);
      viewer.evPostRenderListener(() => {
        try {
          if (this.mapListerTime) {
            this.mapListerTime = false;
            setTimeout(() => {
              this.mapListerTime = true;
              let position = viewer.camera.position;
              var ellipsoid = viewer.scene.globe.ellipsoid;
              var cartesian3 = new Cesium.Cartesian3(position.x, position.y, position.z);
              var cartographic = ellipsoid.cartesianToCartographic(cartesian3);
              var lat = Cesium.Math.toDegrees(cartographic.latitude);
              var lng = Cesium.Math.toDegrees(cartographic.longitude);
              if (lat !== this.areacode.mapLat || lng !== this.areacode.mapLng) {
                this.areacode.mapLng = lng;
                this.areacode.mapLat = lat;
                businessApi.tiandituApi({
                  url: this.$api.mapApi,
                  params: {
                    lat: lat,
                    lon: lng
                  }
                }).then(data => {
                  if (data.result.addressComponent.province !== "福建省") {
                    this.areacode.mapProvince = ""
                    this.areacode.province = "";
                    this.areacode.city = "";
                    this.areacode.town = "";
                    this.areacode.mapCity = ""
                    this.areacode.mapTown = ""
                    this.areacode.mapProvince = ""
                  } else {
                    this.areacode.mapProvince = data.result.addressComponent.province;
                    this.areacode.province = this.$areacode.province[0].areacode;
                    for (let i = 0; i < this.areacode.opt_cities.length; i++) {
                      if (this.areacode.opt_cities[i].name == data.result.addressComponent.city) {
                        this.areacode.city = this.areacode.opt_cities[i].areacode;
                        this.areacode.opt_towns = this.areacode.opt_cities[i].county;
                        this.areacode.mapCity = data.result.addressComponent.city
                        break;
                      }
                    }
                    this.areacode.town = "";
                    for (let i = 0; i < this.areacode.opt_towns.length; i++) {
                      if (this.areacode.opt_towns[i].name == data.result.addressComponent.county) {
                        this.areacode.town = this.areacode.opt_towns[i].areacode;
                        this.areacode.mapTown = data.result.addressComponent.county
                        break;
                      }
                    }
                  }
                  // this.areacode.mapCity =data.result.addressComponent.city
                  // this.areacode.mapTown =data.result.addressComponent.county
                  // this.areacode.mapProvince = data.result.addressComponent.province;
                })
              }
            }, 2000);
          }
        } catch (error) {
          console.log(erroe)
        }


        viewer.getMapScale(obj => {
          this.mapObj.scale = `1:${obj}`;
        });
      })
    }
  },
  created () {

  },
  mounted () {

  },
}
</script>
<style lang='less' scoped>
</style>