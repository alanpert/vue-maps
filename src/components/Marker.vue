<template>
<div>
<!-- MARKER -->
</div>
</template>

<script>
export default {
  props: {
    moptions: {},
    vmap: {}
  },
  data() {
    return {
      thisMarker: {},
      geocoder: {}
    }
  },
  created() {
    // Init
    this.thisMarker = new google.maps.Marker(this.moptions);
    this.addtoCluster();
    this.thisMarker.setPosition(this.moptions.position);
    this.thisMarker.setMap(this.vmap);
    // Pega o nome da localidade
    this.getName();

    // Listeners
    this.thisMarker.addListener('drag', function() {
      this.moveMarker();
    }.bind(this));

    this.thisMarker.addListener('click', function() {
      this.clickMarker();
    }.bind(this));
  },
  methods: {
    moveMarker() {
      // Atualiza posição do marker ao mover
      this.moptions.position.lat = this.thisMarker.getPosition().lat();
      this.moptions.position.lng = this.thisMarker.getPosition().lng();
      this.getName();
    },
    clickMarker() {
      // Envia para o Parent a chamada da Função ao clicar com o esquerdo no Marker
      // - Mostra a InfoWindow -
      this.$parent.$emit('clickMarker', this.thisMarker);
    },
    addtoCluster() {
      // Envia para o Parent a chamada da Função ao criar o Marker
      // - Para adicionar ao Clusterobj -
      this.$parent.$emit('addtoCluster', this.thisMarker);
    },
    getName() {
      // Mostra nome do Local em que o Marker está
      var $this = this;
      var stringCoordinates = this.moptions.position.lat + "," + this.moptions.position.lng;
      var latlngStr = stringCoordinates.split(',', 2);
      var latlng = {lat: parseFloat(latlngStr[0]), lng: parseFloat(latlngStr[1])};
      this.geocoder = new google.maps.Geocoder;
      this.geocoder.geocode({'location': latlng}, function(results, status) {
        if (status === 'OK') {
          if (results[1]) {
            $this.thisMarker.locationname = results[1].formatted_address;
            $this.moptions.locationname = $this.thisMarker.locationname;
          }
        }
      });
    }
  }
}
</script>

<style lang="scss">
</style>
