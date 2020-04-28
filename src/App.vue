<template>
  <div class="wrapper">
    <h1 class="title">Traffic Manager Display</h1>
    <div class="body-wrapper">
      <div class="map-wrapper">
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
                  <li>
                    Status: {{ message.PositionReport.NavigationalStatus }}
                  </li>
                  <li>Vessel type: {{ message.StaticData.VesselType }}</li>
                </ul>
              </div>
            </l-popup>
          </l-marker>
          <l-tile-layer :url="url" :attribution="attribution" />
        </l-map>
      </div>
      <div class="options-wrapper">
        <div class="statistics-wrapper">
          <h2>AIS statistics</h2>
          <ul v-if="this.aisStatistics" >
            <li>
              <h3>Destinations:</h3>
              <ul class="destinations-container">
                <li
                  v-for="destination in this.aisStatistics.destinations"
                  :key="destination"
                >
                  {{ destination }}
                </li>
              </ul>
            </li>
            <li>
              <h3>Average SOG:</h3>
              {{ this.aisStatistics.averageSOG }}
            </li>
            <li>
              <h3>Total Moored:</h3>
              {{ this.aisStatistics.totalMoored }}
            </li>
            <li>
              <h3>Total Underway</h3>
              {{ this.aisStatistics.totalUnderway }}
            </li>
            <li>
              <h3>Vessel Types:</h3>
              <ul class="vessel-container">
                <li
                  v-for="vessel in Object.keys(
                    this.aisStatistics.vesselTypeCount
                  )"
                  :key="vessel"
                >
                  {{ vessel }}: {{ aisStatistics.vesselTypeCount[[vessel]] }}
                </li>
              </ul>
            </li>
            <li>Total Vessels: {{ this.aisStatistics.totalVessels }}</li>
          </ul>
        </div>

        <div class="vessel-selection-wrapper" v-if="vessels.length !== 0">
          <h2>Vessel List</h2>
          <select v-model="selectedVessel" @change="onVesselChange($event)">
            <option :value="''">Any</option>
            <option v-for="v in vessels" :key="v" :value="v">
              {{ v }}
            </option>
          </select>
        </div>
      </div>
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
    this.fetchMessageInterval = setInterval(() => {
      this.fetchAISMessages();
      this.fetchAISStatistics();
    }, 5000);
  },

  beforeDestroy() {
    console.log("Destroying AIS message fetching interval:");
    clearInterval(this.fetchMessageInterval);
  },

  data() {
    return {
      selectedVessel: "",
      vessels: [],
      aisStatistics: null,
      fetchMessageInterval: null,
      messages: [],
      zoom: 10,
      center: latLng(55.66, 12.7875),
      url: "https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png",
      attribution:
        '&copy; <a href="http://osm.org/copyright">OpenStreetMap</a> contributors',
      currentZoom: 10,
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
          let vesselList = [];
          for (let message of messages) {
            if (message.StaticData.Name === this.selectedVessel) {
              this.center = latLng(
                message.PositionReport.Position.coordinates[0],
                message.PositionReport.Position.coordinates[1]
              );
              this.currentCenter = latLng(
                message.PositionReport.Position.coordinates[0],
                message.PositionReport.Position.coordinates[1]
              );
            }
            vesselList.push(message.StaticData.Name);
          }
          this.vessels = vesselList;
          this.messages = messages;
        });
    },
    fetchAISStatistics() {
      fetch("http://localhost:3000/ais-statistics")
        .then((res) => res.json())
        .then((statistics) => {
          this.aisStatistics = statistics;
        });
    },
    onVesselChange(e) {
      if (this.selectedVessel !== "") {
        let message = this.messages.find(
          (message) => message.StaticData.Name === e.target.value
        );
        if (this.zoom !== 15) {
          setTimeout(() => {
            this.currentZoom = 15;
            this.zoom = 15;
          }, 250);
        }
        this.center = latLng(
          message.PositionReport.Position.coordinates[0],
          message.PositionReport.Position.coordinates[1]
        );
        this.currentCenter = latLng(
          message.PositionReport.Position.coordinates[0],
          message.PositionReport.Position.coordinates[1]
        );
      } else {
        this.center = latLng(55.66, 12.7875);
        this.currentCenter = latLng(55.66, 12.7875);
        this.currentZoom = 10;
        this.zoom = 10;
      }
    },
  },
};
</script>

<style>
@import "~leaflet/dist/leaflet.css";

.vessel-container {
  height: 18vh;
  overflow-y: scroll;
}

.destinations-container {
  height: 18vh;
  overflow-y: scroll;
}

.wrapper {
  height: 100vh;
  width: 100vw;
}

.options-wrapper {
  display: flex;
  flex-direction: row;
  align-items: flex-start;
  width: 50%;
  justify-content: space-evenly;
}

.statistics-wrapper {
  border: 1px solid black;
  padding: 55px;
  height: 68vh;
  overflow-y: scroll;
}

.statistics-wrapper h2 {
  font-weight: 300;
  text-align: center;
  margin-bottom: 5vh;
}

.statistics-wrapper ul {
  list-style: none;
  padding: 0;
}

.statistics-wrapper ul li {
  margin-bottom: 23%;
  text-align: center;
}

.map-wrapper {
  height: 100vh;
  width: 60vw;
}

.body-wrapper {
  display: flex;
  flex-direction: row;
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
