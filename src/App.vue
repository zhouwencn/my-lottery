<template>
  <div id="app">
    <Lottery
      @startTurn="start"
      :toRotate="toRotate"
      :winPrize="winPrize"
      :circle="6"
      :duration="4"
      @showPrizePopup="showPrizePopup"
    />
  </div>
</template>

<script>
import Lottery from "./components/Lottery.vue";

export default {
  name: "App",
  components: {
    Lottery,
  },
  data() {
    return {
      prizeList: [],
      toRotate: false,
      winPrize: {}
    }
  },
  created() {
    // 这里可以掉接口拿到奖品列表prizeList并传给子组件
  },
  methods: {
    start() {
      new Promise((resolve) => {
        // 子组件点击抽奖，之后调用接口请求拿到中将奖品传给子组件， toRotate变为true
        setTimeout(() => {
          this.winPrize.prizeId = 5;
          this.toRotate = true;
          resolve();
        }, 1000);
      });
    },
    showPrizePopup() {
      alert("抽到了");
      this.toRotate = false;
    },
  },
};
</script>

<style>
html,
body {
  margin: 0;
  padding: 0;
  background-color: #ccc;
}
</style>
