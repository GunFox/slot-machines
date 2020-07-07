<template>
  <div id="app">
    <div class="slotMhchineBox">
      <slotMachines ref="slotMachines" :lotterynumnber="lotteryNumber" @update="updateLottryNums"/>
    </div>
    <button @click="handleRandomNumbers">随机</button>
    <button @click="handleResetNumbers">重置</button>
  </div>
</template>

<script>
import slotMachines from './components/slotMachines.vue'

export default {
  name: 'App',
  data() {
    return {
      lotteryNumber: "",
    }
  },
  components: {
    slotMachines
  },
  methods: {
    handleRandomNumbers() {
      this.$refs.slotMachines.numbersByRandom()
    },
    handleResetNumbers() {
      this.$refs.slotMachines.resetNumbers();
    },
    updateLottryNums(numbers) {
          this.lotteryNumber = numbers;
    },
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
button  {
  font-size: 18px;
  background: cornflowerblue;
  text-align: center;
  line-height: 40px;
  border: none;
  color: #fff;
  margin-top: 20px;
  outline: none;
  border-radius: 8px;
  box-shadow: 1px 1px 10px rgba(0,0,0,.3);
}
</style>
