<template>
<div>
<!-- InfoWindow -->
</div>
</template>

<script>
export default {
  props: {
    vmap: {},
    infoopen: false,
    linkedmarker: {}
  },
  data() {
    return {
      infoWindow: {}
    }
  },
  created() {
    console.log("InfoWindow Created");
    var contentString = '<div>'+
     '<div>'+
     '</div>'+
     '<h1 class="firstHeading">Title</h1>'+
     '<div>'+
     '<p>Description' +
     '</p>'+
     '</div>'+
     '</div>';
    this.infoWindow = new google.maps.InfoWindow({
      content: contentString
    });
    if (this.infoopen) {
      var $this = this;
      this.openInfoWindow();
    }
    // Listener Botão Close
    this.infoWindow.addListener('closeclick', function() {
      this.$parent.$emit('closeInfoWindow');
    }.bind(this));
  },
  watch: {
    infoopen: function(val) {
      if (val) {
        this.openInfoWindow();
      } else {
        this.infoWindow.close();
      }
    },
    linkedmarker: function() {
      this.openInfoWindow();
    }
  },
  methods: {
    openInfoWindow() {
      this.infoWindow.open(this.vmap, this.linkedmarker);
    }
  }
}
</script>

<style lang="scss">
</style>
