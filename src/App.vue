<template>
  <!-- punto d'entrata -->
  <div id="app">
    <!-- recupero il contenuto del mio componente -->
    <MyMap></MyMap>
  </div>
</template>

<script>

import MyMap from './components/MyMap.vue'

export default {
  name: 'App',
  components: {
    MyMap
  }
}
</script>

<style>
/* INIZIALIZZAZIONE */
* {
  padding: 0;
  margin: 0;
  box-sizing: border-box;
}
html {
  font-family: 'Roboto', sans-serif;
  font-size: 20px;
  font-weight: 400;
}
/* CONTAINER GENERALE */
div#app {
  color: #1D8CAB;
  background-color: #1D8CAB;
}
/* MEDIA-QUERY */
@media screen and (max-width: 600px) {
  html {
    font-size: 16px;
  }
}
@media screen and (max-width: 400px) {
  html {
    font-size: 14px;
  }
}
</style>
