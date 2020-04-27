<template>
  <div class="wrapper">
    <h1 class="title">Traffic Manager Display</h1>
    <l-map
      v-if="showMap"
      :zoom="zoom"
      :center="center"
      :options="mapOptions"
      style="height: 80%"
      @update:center="centerUpdate"
      @update:zoom="zoomUpdate"
    >
      <l-marker
        v-for="message in filteredMessages"
        :key="message._id"
        :lat-lng="message.PositionReport.Position.coordinates"
      >
        <l-popup>
          <div class="vessel-popup">
            <h2>{{ message.StaticData.Name }}</h2>
            <ul>
              <li>Class: {{ message.Class }}</li>
              <li>MMSI: {{ message.MMSI }}</li>
              <li>IMO: {{ message.StaticData.IMO }}</li>
              <li>Destination: {{ message.StaticData.Destination }}</li>
              <li>ETA: {{ message.ETA }}</li>
              <li>Status: {{ message.PositionReport.NavigationalStatus }}</li>
              <li>Vessel type: {{ message.StaticData.VesselType }}</li>
            </ul>
          </div>
        </l-popup>
      </l-marker>
      <l-tile-layer :url="url" :attribution="attribution" />
    </l-map>
    <div class="vessel-selection-wrapper" v-if="vessels.length !== 0">
      <h2>Vessel List</h2>
      <select v-model="selectedVessel">
        <option :value="''">Any</option>
        <option v-for="v in vessels" :key="v" :value="v">
          {{ v }}
        </option>
      </select>
    </div>
  </div>
</template>

<script>
import { latLng } from "leaflet";
import { LMap, LTileLayer, LMarker, LPopup } from "vue2-leaflet";

export default {
  name: "Example",
  components: {
    LMap,
    LTileLayer,
    LMarker,
    LPopup,
  },

  created() {
    console.log("Creating AIS message fetching interval:");
    this.fetchMessageInterval = setInterval(
      () => this.fetchAISMessages(),
      5000
    );
  },

  beforeDestroy() {
    console.log("Destroying AIS message fetching interval:");
    clearInterval(this.fetchMessageInterval);
  },

  data() {
    return {
      selectedVessel: "",
      vessels: [],
      fetchMessageInterval: null,
      messages: [],
      zoom: 12,
      center: latLng(55.66, 12.7875),
      url: "https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png",
      attribution:
        '&copy; <a href="http://osm.org/copyright">OpenStreetMap</a> contributors',
      currentZoom: 12,
      currentCenter: latLng(55.66, 12.7875),
      showParagraph: false,
      mapOptions: {
        zoomSnap: 0.05,
      },
      showMap: true,
    };
  },

  computed: {
    filteredMessages: function() {
      return this.messages.filter((message) => {
        if (this.selectedVessel === "") {
          return true;
        }
        return message.StaticData.Name === this.selectedVessel;
      });
    },
  },

  methods: {
    zoomUpdate(zoom) {
      this.currentZoom = zoom;
    },
    centerUpdate(center) {
      this.currentCenter = center;
    },
    fetchAISMessages() {
      fetch("http://localhost:3000/AIS/list")
        .then((res) => res.json())
        .then((messages) => {
          this.vessels = messages.map((message) => message.StaticData.Name);
          this.messages = messages;
        });
    },
  },
};
</script>

<style>
@import "~leaflet/dist/leaflet.css";
.wrapper {
  height: 100vh;
  width: 100vw;
}

.title {
  text-align: center;
  font-weight: 300;
}

.vessel-popup ul {
  list-style-type: none;
}

.vessel-popup h2 {
  text-align: center;
  font-weight: 300;
}
</style>
