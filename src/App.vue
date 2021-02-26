<template>
  <Header />
  <Map />
</template>

<script>
import Map from './components/Map/Map.vue'
import Header from './components/Header/Header.vue'

export default {
  name: 'App',
  data(){
    return {
      generalData: "test"
    }
  },

  components: {
    Map,
    Header
  }
}
</script>

<style lang='scss'>
@import url('https://fonts.googleapis.com/css2?family=Poppins:ital,wght@0,300;0,400;0,500;1,300&display=swap');
@import "./assets/scss/elements";

html,
body {
  margin: 0;
  padding: 0;
}

#app {
  font-family: $default-font;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 0;
  
}
</style>
