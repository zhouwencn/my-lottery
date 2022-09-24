<template>
  <div id="app">
    <Lottery
      @startTurn="start"
      :to-rotate="toRotate"
      :win-prize="winPrize"
      :circle="6"
      :duration="4"
      :check-lottery="checkLottery"
      @show-prize-popup="showPrizePopup"
    />
  </div>
</template>

<script>
import Lottery from './components/Lottery.vue'
function getRandomInt(min, max) {
  min = Math.ceil(min)
  max = Math.floor(max)
  return Math.floor(Math.random() * (max - min)) + min //不含最大值，含最小值
}
export default {
  name: 'App',
  components: {
    Lottery
  },
  data() {
    return {
      prizeList: [],
      toRotate: false,
      winPrize: {
        prizeId: 1,
        index: 0 // 增加一个index来防止watch的值没有变，不触发的问题
      }
    }
  },
  created() {
    // 这里可以掉接口拿到奖品列表prizeList并传给子组件
  },
  methods: {
    start() {
      new Promise((resolve) => {
        // 子组件点击抽奖，之后调用接口请求拿到中将奖品传给子组件
        setTimeout(() => {
          resolve({
            prizeId: getRandomInt(1, 6),
            index: Date.now()
          })
        }, 1000)
      }).then((res) => {
        this.winPrize.prizeId = res.prizeId
        this.winPrize.index = res.index
      })
    },
    showPrizePopup() {
      alert('抽到了')
      // this.toRotate = false;
    },
    checkLottery() {
      new Promise((resolve) => {
        setTimeout(() => {
          resolve(true)
        }, 300)
      }).then((res) => {
        this.toRotate = res
      })
    }
  }
}
</script>

<style>
html,
body {
  margin: 0;
  padding: 0;
  background-color: #ccc;
}
</style>
