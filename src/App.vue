<template>
  <div id="app">
    <div class="slotMhchineBox">
      <slotMachines ref="slotMachines"/>
    </div>
    <button @click="handleRandomNumbers">random</button>
  </div>
</template>

<script>
import slotMachines from './components/slotMachines.vue'

export default {
  name: 'App',
  components: {
    slotMachines
  },
  methods: {
    handleRandomNumbers() {
      this.$refs.slotMachines.numbersByRandom()
    }
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
.slotMhchineBox {
  height: 180px;
  overflow: hidden;
  border: 1px solid #000;
}
</style>
