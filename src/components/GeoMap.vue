<template>
  <nav class="navbar navbar-dark" style="background: #b2beb5;">
    <div class="container">
      <select class="form-select" v-model="selected" @change="onSelect">
        <option value="osm" class="bg-dark text-white"
          >OSM with GPS data</option
        >
        <option value="wms" class="bg-dark text-white"
          >World Map via WMS</option
        >
        <!-- <option value="mq">Mapquest</option> -->
      </select>
    </div>
  </nav>

  <div ref="geoMap" style="width: 100%; height: 100%"></div>
</template>

<script>
//Open Street Map
//World Map Service
import { onMounted, computed, ref, watch } from "vue";
import { useGeolocation } from "@/composables/useGeolocation.js";
import { useOpenLayers } from "@/composables/useOpenLayers.js";

import Point from "ol/geom/Point";
import { Map, View } from "ol";
import { fromLonLat } from "ol/proj";
import { Group } from "ol/layer";
import "ol/ol.css";

export default {
  name: "GeoMap",

  setup() {
    const geoMap = ref(null);
    const olMap = ref(null);
    const selected = ref("osm");

    const { coords } = useGeolocation();
    const { iconFeature, layers } = useOpenLayers();

    const currPos = computed(() => ({
      lng: coords.value.longitude,
      lat: coords.value.latitude,
    }));

    // layer groups
    var layersOSM = new Group({
      layers: [layers[0], layers[3]],
    });

    var layersWMS = new Group({
      layers: [layers[2]],
    });

    var layersMQ = new Group({
      layers: [layers[1]],
    });

    const onSelect = () => {
      if (selected.value == "wms") {
        olMap.value.setLayerGroup(layersWMS);
        olMap.value.getView().setZoom(4);
      } else if (selected.value == "osm") {
        olMap.value.setLayerGroup(layersOSM);
        olMap.value.getView().setZoom(14);
      } else {
        olMap.value.setLayerGroup(layersMQ);
        olMap.value.getView().setZoom(4);
      }
    };

    onMounted(() => {
      olMap.value = new Map({}
        layers: layersOSM,
        target: geoMap.value,
        view: new View({
          center: [0, 0],
          zoom: 4,
        }),
      });
    });

    watch(currPos, () => {
      // location update
      olMap.value
        .getView()
        .setCenter(fromLonLat([currPos.value.lng, currPos.value.lat]));

      iconFeature.value.setGeometry(
        new Point(fromLonLat([currPos.value.lng, currPos.value.lat]))
      );
    });

    return { currPos, geoMap, selected, onSelect };
  },
};
</script>
