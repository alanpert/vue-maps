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
      infoWindow: {},
      vcontent: ""
    }
  },
  created() {
    console.log("InfoWindow Created");
    this.contentUpdate();
    this.infoWindow = new google.maps.InfoWindow({
      content: this.vcontent
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
      // Listener para saber se o InfoWindow já está fechado ou aberto
      if (val) {
        this.openInfoWindow();
      } else {
        this.infoWindow.close();
      }
    },
    linkedmarker: function() {
      // Listener caso mude o Marker (caso usuário clique em outro marker)
      this.contentUpdate();
      this.infoWindow.setContent(this.vcontent);
      this.openInfoWindow();
    }
  },
  methods: {
    openInfoWindow() {
      // Abre o InfoWindow
      this.infoWindow.open(this.vmap, this.linkedmarker);
    },
    contentUpdate() {
      // Atualiza conteudo dinâmico
      this.vcontent = '<div class="infowindow">'+
          '<div>'+
          '</div>'+
          '<h1 class="firstHeading">' + this.linkedmarker.title + '</h1>'+
          '<div class="mapimg"><img src="' + this.linkedmarker.imgurl + '"></div>'+
        '</div>';
    }
  }
}
</script>

<style lang="scss">
.infowindow {
  h1 {
    text-align: center;
    margin-bottom: 5px;
  }
  .mapimg {
    img {
      max-width: 70px;
    }
  }
}
</style>
