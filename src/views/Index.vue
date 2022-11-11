<template>
    <div class="main">
      <div class="flex">
        <div id="map" class="basemap"></div>
        <div id="option" style="display: none;">
          <button @click="updateMarker()">Modifier</button>
          <button @click="remove()">Supprimer le marqueur</button>
        </div>
        <button style="width: 100px; height: 200px;" @click="create()">Ajouter</button>
      </div>
  </div>
</template>
<script>

  import axios from "axios";
  import "mapbox-gl/dist/mapbox-gl.css";
  import mapboxgl from "mapbox-gl"

  export default {
    name: 'Index',
    data() {
      return {
        access_token: process.env.VUE_APP_MAP_ACCESS_TOKEN,
        coordinates: [48.856614, 2.3522219],
        markers: [],
        map: '',
        id: ''
      };
    },

    async mounted() {
      mapboxgl.accessToken = this.access_token;

      this.map = new mapboxgl.Map({
        container: "map",
        style: "mapbox://styles/mapbox/streets-v11",
        center: [2.3522219, 48.856614],
        zoom: 8
      });

      this.map.addControl(new mapboxgl.NavigationControl(), "top-right");

      try {
        const res = await axios.get(`https://workshoppart1.herokuapp.com/markers`);
        this.markers = res.data;
      } catch (e) {
        console.error(e);
      }

      for (let i = 0; i < this.markers.length; i++) {
        const marker = new mapboxgl.Marker().setLngLat([this.markers[i].long, this.markers[i].lat]);
        marker._element.id = "my-marker";
        marker.addTo(this.map);
        await marker.getElement().addEventListener('click', async (e) => {
          this.id = this.markers[i].id;
          const option = document.getElementById("option");
          option.style.display = "block";
        });
      }

      const geolocate = new mapboxgl.GeolocateControl({
        positionOptions: {
          enableHighAccuracy: true
        },
        trackUserLocation: true
      });

      this.map.addControl(geolocate, "top-right");
    },

    methods: {
      async updateMarker() {
        alert("Cliquer à l'endroit où le marqueur doit être déplacé");
        this.map.on("click", async (e) => {
          await axios.patch('https://workshoppart1.herokuapp.com/markers/'+this.id, {long: e.lngLat.lng, lat: e.lngLat.lat});
          window.location.reload();
        });
      },
      create() {
        alert("Cliquer à l'endroit où le marqueur doit être créé");
        this.map.on("click", async (e) => {
          await axios.post('https://workshoppart1.herokuapp.com/markers', {long: e.lngLat.lng, lat: e.lngLat.lat});
          window.location.reload();
        });
      },
      async remove() {
        await axios.delete('https://workshoppart1.herokuapp.com/markers/'+this.id);
        window.location.reload();
      }
    }

  }
</script>

<style lang="scss" scoped>
   .basemap {
     width: 100%;
     height: 100%;
   }
  .flex {
    display: flex;
    flex-direction: row;
    height: 700px;
  }
</style>
