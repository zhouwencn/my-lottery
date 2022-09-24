<template>
  <div class="turntable-wrapper">
    <div ref="turntable" class="turntable" :style="rotateRound">
      <canvas id="wheelCanvas" width="1000" height="1000"
        >抱歉， 您的浏览器不支持</canvas
      >
    </div>
    <img
      @click="startTurntable"
      class="turntable-btn"
      :src="require('../assets/images/btn.png')"
    />
  </div>
</template>
<script>
let total = 0
let needRotate = 0
export default {
  name: 'lottery',
  props: {
    prizeList: {
      type: Array,
      default() {
        return [
          {
            id: 1,
            image: require('../assets/images/car.png'),
            name: '1',
            remark: 'test'
          },
          {
            id: 2,
            image: require('../assets/images/coupon.png'),
            name: '2',
            remark: 'test'
          },
          {
            id: 3,
            image: require('../assets/images/package.png'),
            name: '3',
            remark: 'test'
          },
          {
            id: 4,
            image: require('../assets/images/packet.png'),
            name: '4',
            remark: 'test'
          },
          {
            id: 5,
            image: require('../assets/images/phone.png'),
            name: '5',
            remark: 'test'
          },
          {
            id: 6,
            image: require('../assets/images/phone.png'),
            name: '6',
            remark: 'test'
          }
        ]
      }
    },
    winPrize: {
      type: Object,
      default() {
        return {
          prizeId: 1
        }
      }
    },
    toRotate: {
      type: Boolean,
      default: false
    },
    // circle圈数越多，duration持续时间越短，转的越快
    // circle圈数越少，duration持续时间越长，转的越慢
    circle: {
      type: Number,
      default: 3
    },
    duration: {
      type: Number,
      default: 5
    },
    checkLottery: {
      type: Function
    }
  },
  data() {
    return {
      rotateStyle: {},
      rotateRound: {}
    }
  },
  created() {
    this.checkLottery()
  },
  mounted() {
    // 监听转盘的webkitTransitionEnd事件
    this.$refs.turntable.addEventListener(
      'webkitTransitionEnd',
      this.popWinInfo,
      false
    )
    this.drawing()
  },
  methods: {
    //开启转盘
    startTurntable() {
      // 去修改this.toRotate的值为true,并请求后端抽奖的结果
      if (!this.toRotate) {
        alert('对不起你没有抽奖资格')
        return
      }
      this.$emit('startTurn')
    },
    //弹出中奖弹框
    popWinInfo() {
      this.$emit('showPrizePopup')
    },
    drawing() {
      const canvas = document.querySelector('#wheelCanvas')
      const ctx = canvas.getContext('2d')
      // canvas自带width和height属性可以理解为分辨率，canvas里面只要涉及半径,translate等之类的操作
      // 只和分辨率有关，和style样式中的width和height没有任何关系
      const width = canvas.width
      const height = canvas.height
      // 计算出每一份应该旋转的角度
      const baseAngle = (Math.PI * 2) / len
      this.drawTurnTableBackGround(ctx, width, height, baseAngle)
      this.drawImgAndText(ctx, width, height, baseAngle)
    },
    drawTurnTableBackGround(ctx, width, height, baseAngle) {
      const { len } = this
      // ctx.clearRect(0, 0, width, height); //去掉背景默认的黑色，可写可不写
      ctx.strokeStyle = 'RGBA(249, 228, 178, 1)' //设置画图线的颜色，这样用扇形拼出来的园的一些弧线或者半径就不会有一些黑色的线条
      const colors = ['RGBA(255, 243, 211, 1)', 'RGBA(255, 250, 237, 1)']
      // 移动原点到圆心
      ctx.translate(width / 2, height / 2)
      // ctx.moveTo(0, 0)
      // ctx.arc(0, 0, width/ 2,  0, Math.PI / 4)
      // ctx.fillStyle = 'red'
      // ctx.stroke()
      // ctx.fill()
      for (let i = 0; i < len; i++) {
        ctx.save()
        ctx.beginPath()
        ctx.moveTo(0, 0)
        // 把扇形向左移动90+baseAngle / 2度，抽奖指示器真好对着每个扇形的中间部分
        // 这样处理的话就不用管rotate旋转了,因为startAngle和endAngle两个都是动态变化的
        ctx.arc(
          0,
          0,
          width / 2,
          baseAngle * i - Math.PI / 2 - baseAngle / 2,
          baseAngle * (i + 1) - Math.PI / 2 - baseAngle / 2,
          false
        )
        if (i % 2 == 0) {
          ctx.fillStyle = colors[0] //设置每个扇形区域的颜色
        } else {
          ctx.fillStyle = colors[1] //设置每个扇形区域的颜色
        }
        ctx.closePath()
        ctx.stroke()
        ctx.fill()
        ctx.restore()
      }
    },
    drawImgAndText(ctx, width, height, baseAngle) {
      const { len } = this
      // 设置填充文字和图片的属性
      ctx.textAlign = 'center'
      ctx.textBaseline = 'middle'
      ctx.font = '60px PingFangSC-Semibold' //设置字号字体
      ctx.fillStyle = 'rgba(255, 67, 93, 1)'

      // 设置文字,图片距离原点中心的距离
      // 同时设置了一个60像素的padding可理解为
      // 但因为的设置的canvas的分辨率比较大,长宽为1000，但是canvas中style样式设置的比较小，才360px
      // 所以canvas在页面只显示360px大小，看起来比较小，但是像素比较高
      // 所以这里减去的分辨率数值得大一些，比如这是设置了60
      // 重要，只要涉及到canvas里面的属性的，比如宽高，半径，translate之类只考虑canvas的分辨率，和style样式没有任何关系
      const FONT_IMG_WIDTH = width / 2 - 60

      // 设置图片的宽高
      const imgWidth = 164
      const imgHeight = 164
      for (let index = 0; index < len; index++) {
        let angle = index * baseAngle
        let img = new Image()
        img.src = this.prizeList[index].image
        img.onload = () => {
          ctx.save()
          ctx.beginPath()
          ctx.rotate(angle)
          // 填充文字和图片
          ctx.fillText(this.prizeList[index].name, 0, -FONT_IMG_WIDTH)
          ctx.fillText(this.prizeList[index].remark, 0, -(FONT_IMG_WIDTH - 80))
          // -imgWidth / 2，x轴移动一半，为了居中
          ctx.drawImage(
            img,
            -imgWidth / 2,
            -(FONT_IMG_WIDTH - 120),
            imgWidth,
            imgHeight
          )
          ctx.closePath()
          ctx.restore()
        }
      }
    }
  },
  computed: {
    len() {
      return this.prizeList.length
    }
  },
  filters: {
    // 处理超长名字
    formatName(name, len) {
      return name.length >= len ? name.slice(0, len) + '...' : name
    }
  },
  watch: {
    prizeList(val) {
      if (val.length > 0) {
        this.drawing()
      }
    },
    winPrize: {
      handler(nval) {
        console.log(nval)
        const prizeId = nval.prizeId
        // 因为抽奖之后不用回正可以连续点击抽奖，所以每次抽奖之前先做一下回正操作
        total -= needRotate
        if (prizeId >= 0) {
          const { len, circle, duration } = this
          const index = this.prizeList.findIndex((item) => item.id === prizeId)
          needRotate = 360 - index * (360 / len)
          total += 360 * circle + needRotate
          this.rotateRound = {
            transform: `rotate(${total}deg)`,
            transition: `all ${duration}s`
          }
        } else {
          alert('抽奖失败')
        }
      },
      deep: true
    }
  }
}
</script>
<style>
/* turntable-wrapper 宽高被子元素撑开，无需设置宽高*/
.turntable-wrapper {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  /*background-color: #ed2222;*/
  font-family: PingFang SC;
}
/* 绝对定位，按钮放到中间 */
.turntable-btn {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: 120px;
  height: 120px;
}
.turntable {
  width: 380px;
  height: 380px;
  position: relative;
}
/*canvas居中，并设置宽高*/
#wheelCanvas {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: 360px;
  height: 360px;
  border-radius: 50%;
  /*background-color: blue;*/
}
</style>
