<template>
<div id="mapcontainer">

  <div id="mapg">
    <div class="">
      <v-infowindow v-if="infoWindow.created" :vmap="map" :infoopen="infoWindow.show" :linkedmarker="infoWindow.linkedMarker"></v-infowindow>
    </div>
    <div>
      <v-marker v-for="m in markers" :moptions="m" :vmap="map" :locationname="locationname"></v-marker>
    </div>
  </div>

  <p>----------</p>
  <table>
    <tr v-for="m in markers">
      <td><input type="number" v-model="m.position.lat" disabled></td>
      <td><input type="number" v-model="m.position.lng" disabled></td>
      <td><p>Name: {{m.locationname}}</p></td>
    </tr>
  </table>

</div>
</template>

<script>
import vMarker from './Marker.vue'
import vInfowindow from './InfoWindow.vue'

export default {
  data() {
    return {
      map: {},
      markers: [],
      mapOptions: {
        center: { lat: 48.8538302, lng: 2.2982161 },
        scrollwheel: false,
        zoom: 8
      },
      markerClusterobj: {},
      infoWindow: {
        created: false,
        show: false,
        linkedMarker: {}
      }
    }
  },
  created() {
    // Init
    var $this = this;
    setTimeout(function () {
      $this.vloadmap();
    }, 200);
    this.setEmitListeners();
  },
  filters: {

  },
  computed: {

  },
  methods: {
    setEmitListeners() {
      // Cria Listeners para os Eventos enviados dos Childs Components
      this.$on('addtoCluster', this.addtoCluster);
      this.$on('clickMarker', this.showInfoWindow);
      this.$on('closeInfoWindow', this.closeInfoWindow);
    },
    vloadmap() {
      // Carrega e Inicia o API do Google Maps
      console.log("LoadMap");
      this.map = new google.maps.Map(document.getElementById('mapg'), this.mapOptions);
      this.map.addListener('rightclick', this.mapRclick);
      this.markerClusterobj = new MarkerClusterer(this.map, this.markerClusterobj, {imagePath: 'https://developers.google.com/maps/documentation/javascript/examples/markerclusterer/m'});
    },
    mapRclick(e) {
      // Click Botão Direito no Mapa para criar novo Marker
      console.log("Map - RightCick");
      const createdMarker = this.addMarker(e.latLng.lat(), e.latLng.lng());
    },
    addMarker(mlat,mlng) {
      // Add Marker no Mapa
      // - Cria um novo <v-marker> -
      console.log("AddMarker");
      this.markers.push({
        position: { lat: mlat, lng: mlng},
        title: "teste",
        opacity: 1,
        draggable: true,
        enabled: true,
        locationname: ""
      });
      return this.markers[this.markers.length - 1]
    },
    addtoCluster(objMarker) {
      // Adicionando o Marker Criado no obj markerClusterobj
      // - ao dar zoom out, os markers próximos se juntam -
      this.markerClusterobj.addMarker(objMarker);
    },
    showInfoWindow(objMarker) {
      // Montando InfoWindow Ao clicar no Marker

      // Iniciando o InfoMarker - Primeira vez que clicar em um marker
      if (!this.infoWindow.created) {
        this.infoWindow.created = true
        this.infoWindow.linkedMarker = objMarker;
      }

      // Toggle Info Window
      if (objMarker == this.infoWindow.linkedMarker) {
        this.infoWindow.show ? this.infoWindow.show = false : this.infoWindow.show = true;
      } else {
        this.infoWindow.linkedMarker = objMarker;
      }
    },
    closeInfoWindow() {
      this.infoWindow.show = false;
    }
  },
  components: {
    vMarker,
    vInfowindow
  }
}

</script>

<style lang="scss">
#mapg {
  display: block;
  width: 100%;
  height: 500px;
}
</style>
