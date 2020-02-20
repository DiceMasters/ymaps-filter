<template>
  <div id="app">
    <div class="wrap">
      <GeoSelect
        id="suggest"
        label="Адрес"
        :initial="initialGeoSelectValue"
        v-model="parentValue"
      />
      <p style="color: white">parentValue {{ parentValue }}</p>
    </div>
  </div>
</template>

<script>
import GeoSelect from './components/GeoSelect'

export default {
  name: 'App',
  components: {
    GeoSelect
  },
  data() {
    return {
      initialGeoSelectValue: 'Реутовское шоссе',
      parentValue: ''
    }
  }
}
</script>

<style lang="scss">
html, body {
  min-width: 100vw;
  min-height: 100vh;
  padding: 0;
  margin: 0;
  overflow-x: hidden;
}

#app {
  width: 100vw;
  height: 100vh;
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;

  .wrap {
    width: 100%;
    height: calc(100% - 40px);
    background: #C04848;  /* fallback for old browsers */
    background: -webkit-linear-gradient(to right, #480048, #C04848);  /* Chrome 10-25, Safari 5.1-6 */
    background: linear-gradient(to right, #480048, #C04848); /* W3C, IE 10+/ Edge, Firefox 16+, Chrome 26+, Opera 12+, Safari 7+ */
    display: flex;
    flex-direction: column;
    justify-content: flex-start;
    align-items: center;
    padding-top: 40px;
  }
}
</style>
