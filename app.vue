<script setup>
import { onMounted, ref } from 'vue';
import mapboxgl from "mapbox-gl";
import "mapbox-gl/dist/mapbox-gl.css";
import style from "./mapbox/style.json";

const hoverFeature = ref();
const legendItems = ref([]);
const attribution = ref();
const mapEl = ref();

mapboxgl.accessToken = 'pk.eyJ1Ijoicmdhc3RvbiIsImEiOiJJYTdoRWNJIn0.MN6DrT07IEKXadCU8xpUMg';

onMounted(() => {
  const map = new mapboxgl.Map({
    "container": mapEl.value,
    "style": style,
    "customAttribution": attribution.value.innerHTML
  });

  map.on("load", async () => {
    const neighborhoods = await import("../data/count-by-neighborhoods.json");
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

    map.on("mouseleave", layerName, (event) => {
      hoverFeature.value = null;

      map.removeFeatureState({
        source: sourceName
      });
    });

    const fillColorStyle = map.getPaintProperty(layerName, "fill-extrusion-color");
    let fromValue = 0;
    fillColorStyle.splice(0, 2);

    for (let index = 0; index < fillColorStyle.length; index += 2) {
      const toValue = fillColorStyle[index + 1];
      const item = {
        color: fillColorStyle[index]
      };

      if (index == fillColorStyle.length - 1) {
        item.text = `>=${fromValue}`;
      } else {
        item.text = `${fromValue}-${toValue - 1}`;
      }

      legendItems.value.push(item);
      fromValue = toValue;
    }
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
      <div class="panel floating bottom" v-show="legendItems.length > 0">
        <div># of Sites</div>
        <div v-for="item in legendItems">
          <span class="color" v-bind:style="{ backgroundColor: item.color }"></span>
          &nbsp;
          <span>{{ item.text }}</span>
        </div>
      </div>
    </div>
  </div>
  <div ref="attribution" class="hidden">
    <a href='https://hpla.lacity.org/' target='_blank'>
      Historic Places LA
    </a>
    |
    <a href='https://geohub.lacity.org/' target='_blank'>
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