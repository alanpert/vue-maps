<template>
<div id="mapcontainer">

  <section class="map-wrapper clearfix">
    <!-- MAPA -->
    <div id="mapg">
      <!-- INFOWINDOW -->
      <div>
        <v-infowindow v-if="infoWindow.created" :vmap="map" :infoopen="infoWindow.show" :linkedmarker="infoWindow.linkedMarker"></v-infowindow>
      </div>

      <!-- MARKERS -->
      <div>
        <v-marker v-for="m in markers" :moptions="m" :vmap="map" :locationname="locationname"></v-marker>
      </div>
    </div>
    <!-- /MAPA -->

    <!-- JANELA DE INFORMAÇOES NA LATERAL -->
    <div class="infos">
      <h2> {{selectedMarker.title}} </h2>
      <p class="location"> {{selectedMarker.locationname}} </p>
      <div class="img-wrapper">
        <img :src="selectedMarker.imgurl" alt="">
      </div>
      <p class="description">
        {{selectedMarker.description}}
      </p>
    </div>
  </section>

  <!-- FILTROS -->
  <section class="filter">
    <input type="text" name="" value="" placeholder="Search..." v-model="filterinput">
    <div class="item-container" v-for="m in filteredItems" @click="clickItemFilter(m)">
      <h3>{{m.title}}</h3>
      <p>{{m.locationname}}</p>
    </div>
  </section>

</div>
</template>

<script>
import vMarker from './Marker.vue'
import vInfowindow from './InfoWindow.vue'

export default {
  data() {
    return {
      jsonURL: "/dist/data/markers.json",
      map: {},
      markers: [],
      mapOptions: {
        center: { lat: 48.8538302, lng: 2.2982161 },
        scrollwheel: false,
        zoom: 8,
        streetViewControl: false,
      },
      markerClusterobj: {},
      infoWindow: {
        created: false,
        show: false,
        linkedMarker: {}
      },
      selectedMarker: {},
      filterinput: ""
    }
  },
  created() {
    // Init
    var $this = this;
    setTimeout(function () {
      $this.vloadmap();
      $this.loadjson();
    }, 200);
    this.setEmitListeners();
  },
  filters: {

  },
  computed: {
    filteredItems() {
      // Filtro para mostrar os itens ao Digitar no campo Input (Filtra pelo nome e pela localidade)
      var $this = this;
      return this.markers.filter(function(item) {
        return item.title.toLowerCase().indexOf($this.filterinput.toLowerCase()) > -1 || item.locationname.toLowerCase().indexOf($this.filterinput.toLowerCase()) > -1
      })
    }
  },
  methods: {
    setEmitListeners() {
      // Cria Listeners para os Eventos enviados dos Childs Components
      this.$on('addtoCluster', this.addtoCluster);
      this.$on('clickMarker', this.showInfos);
      this.$on('closeInfoWindow', this.closeInfoWindow);
    },
    vloadmap() {
      // Carrega e Inicia o API do Google Maps
      console.log("---LoadMap");
      this.map = new google.maps.Map(document.getElementById('mapg'), this.mapOptions);
      // this.map.addListener('rightclick', this.mapRclick);
      this.markerClusterobj = new MarkerClusterer(this.map, this.markerClusterobj, {imagePath: 'https://developers.google.com/maps/documentation/javascript/examples/markerclusterer/m'});
    },
    loadjson() {
      // Carrega o arquivo "this.jsonURL" com os markers já adicionados
      var $this = this;
      console.log("---Load Json");
      $.ajax({
        url: $this.jsonURL,
        method: 'GET',
        success: function (data) {
          console.log("Load Success");
          for (var i = 0; i < data.length; i++) {
            var datatemp = data[i];
            $this.addMarkerFromJson(datatemp)
          }
        },
        error: function (error) {
          alert(JSON.stringify(error));
        }
      });
    },
    mapRclick(e) {
      // Click Botão Direito no Mapa para criar novo Marker
      console.log("@Map - RightCick");
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
    addMarkerFromJson(mdata) {
      // Add Marker no Mapa a partir do Json Carregado
      // - Cria um novo <v-marker> -
      console.log("AddMarker");
      this.markers.push({
        position: { lat: mdata.position.lat, lng: mdata.position.lng},
        title: mdata.title,
        opacity: 1,
        draggable: false,
        enabled: true,
        locationname: "",
        imgurl: mdata.imgurl,
        description: mdata.description
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
    showInfos(objMarker) {
      this.selectedMarker = objMarker;
      this.showInfoWindow(objMarker);
    },
    closeInfoWindow() {
      // Fecha InfoWindow
      this.infoWindow.show = false;
    },
    clickItemFilter(marker) {
      // Ao clicar em um item do Filtro (Mostra o Marker e abre a InfoWindow)
      console.log("@Click Item Filter");
      this.selectedMarker = marker;
      this.map.setZoom(14);
      this.map.panTo(marker.position);
      this.$emit('openClickedMarkerInfo', marker);
    }
  },
  components: {
    vMarker,
    vInfowindow
  }
}

</script>

<style lang="scss">
#mapcontainer {
  .map-wrapper {
    #mapg {
      display: block;
      width: 70%;
      height: 500px;
      float: left;
    }
    .infos {
      display: block;
      width: 30%;
      height: 500px;
      padding-bottom: 30px;
      position: relative;
      float: left;
      overflow-y: auto;
      color: #353535;
      -webkit-box-shadow: -10px 10px 30px 0px rgba(0,0,0,0.12);
      -moz-box-shadow: -10px 10px 30px 0px rgba(0,0,0,0.12);
      box-shadow: -10px 10px 30px 0px rgba(0,0,0,0.12);
      h2 {
        text-align: center;
        padding: 10px 0;
      }
      .location {
        text-align: center;
        font-size: 14px;
      }
      .img-wrapper {
        padding: 35px;
        img {
          width: 100%;
        }
      }
      .description {
        padding: 0 20px;
      	font-size: 14px;
      	line-height: 20px;
      }
    }
  }
  .filter {
    margin-top: 25px;
    padding-left: 15px;
    input {
    	width: 500px;
      padding: 10px;
      outline: none;
    	color: #797979;
    }
    .item-container {
      width: 500px;
      padding: 15px;
      border-bottom: 1px solid #bebebe;
      cursor: pointer;
      -webkit-transition: all .3s ease-in-out;
      -moz-transition: all .3s ease-in-out;
      -o-transition: all .3s ease-in-out;
      transition: all .3s ease-in-out;
      h3, p {
        -webkit-transition: all .3s ease-in-out;
        -moz-transition: all .3s ease-in-out;
        -o-transition: all .3s ease-in-out;
        transition: all .3s ease-in-out;
      }
      h3 {
      	line-height: 28px;
      }
      p {
        color: #797979;
        font-size: 15px;
      }
      &:hover {
        background: #f3f3f3;
        h3 {
          color: #4c6bb7;
        }
        p {
          color: #7e97d6;
        }
      }
    }
  }
}

</style>
