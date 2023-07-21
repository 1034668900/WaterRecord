<script>
export default {
  data() {
    return {
      screenData: {
        width: 0,
        height: 0,
      },
      // 完成天数
      completeDays: 0,
      // 杯子倾斜角正切值
      tantAngle: 0,
      // 杯子容积
      cupVolume: 0,
      // 今日喝水目标
      todayTarget: 2.5,
      // 目前已喝
      currentDrink: 0.5,
      prepareDrink: "",
      prepareTargetDrink: "",
    };
  },
  mounted() {
    console.log("@@@0", window.screen);
    // 获取设备屏幕大小
    let { width, height } = window.screen;
    this.screenData.width = width;
    this.screenData.height = height;
    console.log("&&&&&", this.screenData);
    this.drawCup();
  },
  methods: {
    // 绘制杯子
    drawCup() {
      // 数据准备
      let width = this.screenData.width;
      let height = this.screenData.height;
      let startX = width * 0.25;
      let startY = height * 0.5;
      console.log(startX, startY, width, height);
      // 绘制杯子
      const ctx = this.$refs.canvas.getContext("2d");
      ctx.lineWidth = 10;
      ctx.beginPath();
      ctx.strokeStyle = "rgb(54, 57, 56,0.5)";
      ctx.moveTo(startX, startY);
      ctx.lineTo(width * 0.7, startY);
      ctx.lineTo(width * 0.82, height * 0.1);
      ctx.lineTo(width * 0.13, height * 0.1);
      ctx.lineTo(width * 0.252, height * 0.5055);
      ctx.stroke();
      ctx.fillStyle = "rgba(61, 61, 61,0.05)";
      ctx.fill();

      console.log(
        "杯子尺寸",
        "底部",
        width * 0.7 - startX,
        "高",
        startY - 0.1 * height,
        "顶",
        0.82 * width - 0.13 * width
      );
      // 计算杯子面积
      this.cupVolume =
        ((0.7 * width - startX + (0.82 - 0.13) * width) *
          (startY - height * 0.1)) /
        2;
      // 计算倾斜角度
      this.tantAngle =
        ((0.82 - 0.13) * width - (0.7 * width - startX)) /
        2 /
        (startY - height * 0.1);
    },
    // 绘制水
    drawWater() {
      let width = this.screenData.width;
      let height = this.screenData.height;
      let startX = width * 0.25;
      let startY = height * 0.5;
      // 杯子底边
      let bottomLength = 0.7 * width - startX;
      // 获取已喝水的百分比
      let percenTage = this.getDrinkedPercentage();
      // 本次需要绘制的上底
      let topLength = Math.sqrt(
        4 * this.tantAngle * percenTage * this.cupVolume +
          Math.pow(bottomLength, 2),
        2
      );
      // 本次需要绘制的高
      let drawHeight = (topLength - bottomLength) / (2 * this.tantAngle);
      // 左右结束点
      let endRight = startX + bottomLength + (topLength - bottomLength) / 2;
      let endLeft = startX - (topLength - bottomLength) / 2;

      console.log(
        "参数对比",
        "底",
        bottomLength,
        "百分比",
        percenTage,
        "顶",
        topLength,
        "高",
        drawHeight,
        endRight,
        endLeft
      );

      // 绘制水
      const ctx = this.$refs.canvas.getContext("2d");
      ctx.lineWidth = 1;
      ctx.beginPath();
      ctx.strokeStyle = "rgb(54, 57, 56,0.5)";
      ctx.moveTo(startX, startY);
      ctx.lineTo(width * 0.7, startY);
      ctx.lineTo(endRight, startY - drawHeight);
      ctx.lineTo(endLeft, startY - drawHeight);
      ctx.lineTo(startX, startY);
      ctx.stroke();
      ctx.fillStyle = "rgb(16, 116, 167)";
      ctx.fill();
    },
    // 计算已喝水所占百分比
    getDrinkedPercentage() {
      console.log("))))))))))", this.currentDrink / this.todayTarget);
      return this.currentDrink / this.todayTarget;
    },
    // 修改待喝水
    changePrepareDrink(event) {
      // 数据校验
      if (event.target.value.trim().length != 0) {
        let difValue =
          this.currentDrink + (+event.target.value)/1000 - this.todayTarget;
        if (difValue > 0) {
          this.prepareDrink = this.todayTarget;
        } else {
          this.prepareDrink += (+event.target.value)/1000;
        }
        console.log("preada", this.prepareDrink);
      }
    },
    changetodayTarget() {
        this.todayTarget = this.prepareTargetDrink;
        this.prepareTargetDrink = ''
    },
    // 修改待设目标
    changePrepareTargetDrink(event) {
      console.log(
        "GGGGGGGG",
        event.target.value.trim().length,
        event.target.value
      );
      if (event.target.value.trim().length != 0) {
        this.prepareTargetDrink = +event.target.value;
      }
    },
    // 干了这杯
    changeCurrentDrink() {
      this.currentDrink += +this.prepareDrink;
      if (this.currentDrink >= this.todayTarget) {
        this.currentDrink = this.todayTarget;
        this.drawWater();
      } else if (this.currentDrink == 0) {
        return;
      } else {
        this.drawWater();
      }
      this.prepareDrink = ''
      if(this.isCompleteTask()){
        this.completeDays++
      }
    },
    // 今日目标达成
    completeToday() {},
    // 清空水杯
    clearCup() {
      // 清空数据
      this.currentDrink = 0;
      this.prepareDrink = "";
      const ctx = this.$refs.canvas.getContext("2d");
      // 清空画布
      ctx.clearRect(0, 0, 390, 700);
      // 重新绘制水杯
      this.drawCup();
    },
    isCompleteTask(){
      return this.currentDrink == this.todayTarget
    }
  },
};
</script>

<template>
  <div id="app">
    <div class="title">喝水记录</div>
    <div class="cup">
      <!-- 今日目标 -->
      <div class="todayTarget">今日喝水目标：{{ todayTarget }} L</div>
      <div class="todayTarget">今日已喝：{{ currentDrink }} L</div>
      <div class="todayTarget">已完成天数：{{ completeDays }} 天</div>
      <canvas width="380" height="700" ref="canvas"></canvas>
      <div class="button">
        <div class="btn1">
          <button @click="changeCurrentDrink">干了这杯!</button>
          <input
            @change="changePrepareDrink($event)"
            :value="prepareDrink"
            type="number"
            placeholder="这杯准备喝多少呢(mL)"
          />
        </div>
        <div class="btn1">
          <button @click="changetodayTarget">修改今日目标</button>
          <input
            @input="changePrepareTargetDrink($event)"
            :value="prepareTargetDrink"
            type="number"
            placeholder="修改您今日喝水目标(L)"
          />
        </div>
        <div class="btn2">
          <button @click="clearCup">清空水杯</button>
          <button @click="completeToday">今日目标完成</button>
        </div>
      </div>
    </div>

    <div></div>
  </div>
</template>

<style>
* {
  margin: 0;
  padding: 0;
}
html {
  font-size: 10px;
}
div {
  box-sizing: border-box;
}
.title {
  font-size: 4rem;
  text-align: center;
  background: linear-gradient(103.3deg, rgb(252, 225, 208) 30%, rgb(255, 173, 214) 55.7%, rgb(162, 186, 245) 81.8%);

}
.cup {
  position: relative;
  width: 100%;
  background-color: beige;
}
.todayTarget {
  font-size: 2.5rem;
  padding-top: 1rem;
}
.button {
  position: absolute;
  width: 100%;
  bottom: 8rem;
}
.btn1 {
  display: flex;
  width: 100%;
  padding-left: 2rem;
  margin-bottom: 1rem;
}
input{
  border-color: rgb(61, 61, 237);
  padding-left: 1rem;
  border-radius: 1rem;
}
button {
  width: 10rem;
  height: 3.5rem;
  border-radius: 1rem;
  border: none;
  background-image: linear-gradient(110.6deg, rgb(179, 157, 219) 7%, rgb(150, 159, 222) 47.7%, rgb(24, 255, 255) 100.6%);

}
.btn1 button {
  margin-right: 2rem;
}
.btn2 {
  display: flex;
  width: 100%;
  flex-direction: column;
  align-items: left;
  justify-content: center;
  margin-top: 1rem;
  padding-left: 2rem;
}
.btn2 button{
  margin-bottom: 1rem;
}
</style>
