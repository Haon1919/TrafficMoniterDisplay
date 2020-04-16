<template>
  <div id="app">
    <l-map class="map">
      <l-tile-layer></l-tile-layer>
    </l-map>
  </div>
</template>

<script>
// import { LMap, LTileLayer, LMarker } from 'vue2-leaflet';
import { LMap, LTileLayer } from 'vue2-leaflet';


export default {
  name: 'App',
  components: {
    LMap,
    LTileLayer
  },
}
</script>

<style>
.map{
  height: 100%;
  width: 100%;
}
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
  background-color: red;
  height: 100vh;
}
.my-leaflet-map-container img {
    max-height: none;
}
</style>
