<template>
  <CoinPaprika msg="Coinpaprika"/>
</template>

<script>
import CoinPaprika from './components/CoinPaprika.vue'

export default {
  name: 'App',
  components: {
    CoinPaprika
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
