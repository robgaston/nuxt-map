<script setup>
import { onMounted, ref } from "vue";
import mapboxgl from "mapbox-gl";
import "mapbox-gl/dist/mapbox-gl.css";
import style from "./mapbox/style.json";

mapboxgl.accessToken = "pk.eyJ1Ijoicmdhc3RvbiIsImEiOiJJYTdoRWNJIn0.MN6DrT07IEKXadCU8xpUMg";

const attribution = ref();
const mapEl = ref();
const hoverFeature = ref();
const fillColorStyle = ref();

const legendEntries = computed(() => {
  let fromValue = 0;
  const entries = [];
  if (fillColorStyle.value) {
    const styleArray = fillColorStyle.value.toSpliced(0, 2);
    for (let index = 0; index < styleArray.length; index += 2) {
      const toValue = styleArray[index + 1];
      const entry = {
        color: styleArray[index],
        text: (index == styleArray.length - 1) ? `>=${fromValue}` : `${fromValue}-${toValue - 1}`
      };
      entries.push(entry);
      fromValue = toValue;
    }
  }
  return entries;
});

onMounted(() => {
  const map = new mapboxgl.Map({
    "container": mapEl.value,
    "style": style,
    "bounds": [
      -118.668117720454,
      33.70467436671,
      -118.155370349007,
      34.3373108721271
    ],
    "fitBoundsOptions": {
      "pitch": 40,
      "padding": {
        "top": 0,
        "bottom": 100,
        "left": 10,
        "right": 10
      }
    },
    "customAttribution": attribution.value.innerHTML
  });

  map.on("load", async () => {
    const neighborhoods = await import("../data/neighborhoods.json");
    const sourceName = "neighborhoods-polygons";
    const layerName = "neighborhoods-fill";

    map.getSource(sourceName)
      .setData(neighborhoods);

    map.on("mousemove", layerName, (event) => {
      hoverFeature.value = event.features[0];
      map.removeFeatureState({
        source: sourceName
      });
      map.setFeatureState({
        source: sourceName,
        id: event.features[0].id
      }, {
        hover: true
      });
    });

    map.on("mouseleave", layerName, () => {
      hoverFeature.value = null;
      map.removeFeatureState({
        source: sourceName
      });
    });

    fillColorStyle.value = map.getPaintProperty(layerName, "fill-extrusion-color");
  });
});
</script>

<template>
  <div class="container">
    <div class="panel">
      <div class="title">
        Cataloged Historic Sites in Los Angeles by Neighborhood
      </div>
      <div class="subtitle">
        Rob Gaston, March 2024
      </div>
    </div>
    <div class="content">
      <div ref="mapEl" class="map"></div>
      <div class="panel floating top" v-if="hoverFeature">
        <div>{{ hoverFeature.properties.name }}</div>
        <div>
          <span>{{ hoverFeature.properties.count }}</span>
          historic site(s) cataloged
        </div>
      </div>
      <div class="panel floating bottom" v-show="legendEntries.length > 0">
        <div># of Sites</div>
        <div v-for="entry in legendEntries">
          <span class="color" v-bind:style="{ backgroundColor: entry.color }"></span>
          &nbsp;
          <span>{{ entry.text }}</span>
        </div>
      </div>
    </div>
  </div>
  <div ref="attribution" class="hidden">
    <a href="https://hpla.lacity.org/" target="_blank">
      Historic Places LA
    </a>
    |
    <a href="https://geohub.lacity.org/" target="_blank">
      Los Angeles City GeoHub
    </a>
  </div>
</template>

<style scoped>
.container {
  display: flex;
  flex-flow: column;
  height: 100%;
}

.content {
  flex-grow: 1;
  position: relative
}

.map {
  height: 100%;
}

.hidden {
  display: none;
}

.panel {
  border: 1px solid black;
  background-color: rgb(190, 190, 190);
  padding: 15px;
}

.floating {
  position: absolute;
  right: 20px;
}

.top {
  top: 20px;
}

.bottom {
  bottom: 50px;
}

.title {
  font-weight: bold;
}

.subtitle {
  font-size: 0.9em;
}

.color {
  display: inline-block;
  height: 10px;
  width: 10px;
  border: 1px solid black;
}
</style>