<script setup>
import { onMounted, ref } from 'vue'
import mapboxgl from "mapbox-gl";
import "mapbox-gl/dist/mapbox-gl.css";
import custom from "./custom-style.json";

const hoverFeature = ref();
const legendItems = ref([]);

mapboxgl.accessToken = 'pk.eyJ1Ijoicmdhc3RvbiIsImEiOiJJYTdoRWNJIn0.MN6DrT07IEKXadCU8xpUMg';

onMounted(() => {
  const map = new mapboxgl.Map({
    "container": "map",
    "style": "mapbox://styles/rgaston/cltah5jc400kj01raejpg0tkh",
    "zoom": 9,
    "center": [
      -118.3875,
      34.03756
    ],
    "pitch": 40,
    "customAttribution": "<a href='https://hpla.lacity.org/' target='_blank'>Historic Places LA</a> | <a href='https://geohub.lacity.org/' target='_blank'>Los Angeles City GeoHub</a>"
  });

  map.on("load", async () => {
    const neighborhoods = await import("../data/count-by-neighborhoods.json");
    const style = map.getStyle();
    style.sources = {
      ...style.sources,
      ...custom.sources
    };
    style.layers.push(...custom.layers);
    map.setStyle(style);

    map.getSource("neighborhoods-polygons")
      .setData(neighborhoods);

    map.on("mousemove", "neighborhoods-fill", (event) => {
      hoverFeature.value = event.features[0];

      map.removeFeatureState({
        source: "neighborhoods-polygons"
      });
      map.setFeatureState({
        source: "neighborhoods-polygons",
        id: event.features[0].id
      }, {
        hover: true
      });
    });

    map.on("mouseleave", "neighborhoods-fill", (event) => {
      hoverFeature.value = null;

      map.removeFeatureState({
        source: "neighborhoods-polygons"
      });
    });

    const fillColorStyle = map.getPaintProperty("neighborhoods-fill", "fill-extrusion-color");
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
  <header>
    Cataloged Historic Sites in Los Angeles by Neighborhood
    <div>Rob Gaston, March 2024</div>
  </header>
  <main>
    <div id="map"></div>
    <div id="popup" v-if="hoverFeature">
      <div>{{ hoverFeature.properties.name }}</div>
      <div>
        <span>{{ hoverFeature.properties.count }}</span>
        historic site(s) cataloged
      </div>
    </div>
    <div id="legend">
      <div># of Sites</div>
      <div v-for="item in legendItems">
        <span class="color" v-bind:style="{ backgroundColor: item.color }"></span>
        &nbsp;
        <span>{{ item.text }}</span>
      </div>
    </div>
  </main>
</template>

<style scoped>
main {
  display: block;
  position: fixed;
  top: 66px;
  right: 0;
  bottom: 0;
  left: 0;
}

#map {
  height: 100%;
}

header {
  font-weight: bold;
  background-color: rgb(190, 190, 190);
  border-bottom: 2px solid black;
}

header div {
  font-size: 0.9em;
  font-weight: normal;
}

#popup {
  top: 86px;
}

#popup,
#legend {
  position: fixed;
  right: 20px;
  border: 1px solid black;
}

#legend {
  bottom: 40px;
}

header,
#popup,
#legend {
  background-color: rgb(190, 190, 190);
  padding: 15px;
}

.color {
  display: inline-block;
  height: 10px;
  width: 10px;
  border: 1px solid black;
}
</style>