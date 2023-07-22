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
      todayTarget: 0,
      // 目前已喝
      currentDrink: 0,
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
    // 读取本次存储的数据
    let dataRecord = JSON.parse(localStorage.getItem("drinkWaterRecord"));
    if (dataRecord instanceof Object) {
      this.todayTarget = dataRecord.todayTarget;
      this.currentDrink = dataRecord.currentDrink;
      this.completeDays = dataRecord.completeDays;
    }
    if (+this.currentDrink != 0) {
      this.drawWater();
    }
  },
  methods: {
    // 绘制杯子
    drawCup() {
      // 数据准备
      let width = this.screenData.width;
      let height = this.screenData.height;
      let startX = width * 0.25;
      let startY = height * 0.42;
      // 绘制杯子
      this.draw(
        startX,
        startY,
        0.7 * width,
        startY,
        0.82 * width,
        0.1 * height,
        0.13 * width,
        0.1 * height,
        0.252 * width,
        0.4255 * height,
        10
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
      let startY = height * 0.42;
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

      // 绘制
      this.draw(
        startX,
        startY,
        0.7 * width,
        startY,
        endRight,
        startY - drawHeight,
        endLeft,
        startY - drawHeight,
        startX,
        startY,
        1,
        "rgb(16,116,167)"
      );
    },
    draw(
      startX,
      startY,
      firstX,
      firstY,
      secondX,
      secondY,
      thirdX,
      thirdY,
      fourthX,
      fourthY,
      lineWidth = 1,
      rgb = "rgba(61, 61, 61,0.05)"
    ) {
      const ctx = this.$refs.canvas.getContext("2d");
      ctx.lineWidth = lineWidth;
      ctx.beginPath();
      ctx.strokeStyle = "rgb(54, 57, 56,0.5)";
      ctx.moveTo(startX, startY);
      ctx.lineTo(firstX, firstY);
      ctx.lineTo(secondX, secondY);
      ctx.lineTo(thirdX, thirdY);
      ctx.lineTo(fourthX, fourthY);
      ctx.stroke();
      ctx.fillStyle = rgb;
      ctx.fill();
    },
    // 计算已喝水所占百分比
    getDrinkedPercentage() {
      console.log("))))))))))", typeof this.currentDrink);
      return +this.currentDrink / +this.todayTarget;
    },
    // 修改待喝水
    changePrepareDrink(event) {
      // 数据校验
      if (event.target.value.trim().length != 0) {
        this.prepareDrink += +event.target.value / 1000;
      }
    },
    // 修改今日目标
    changetodayTarget() {
      if (+this.prepareTargetDrink != 0) {
        this.todayTarget = +this.prepareTargetDrink;
        this.prepareTargetDrink = "";
        this.clearCup(true);
        this.drawWater();
        this.saveRecord();
      }
    },
    // 修改待设目标
    changePrepareTargetDrink(event) {
      if (event.target.value.trim().length != 0) {
        this.prepareTargetDrink = +event.target.value;
      }
    },
    // 干了这杯
    changeCurrentDrink() {
      // 用于判断这次喝水没
      let oldDrink = +this.currentDrink;
      this.currentDrink += +this.prepareDrink;
      // 处理小数
      this.currentDrink = this.dealDecimasl(this.currentDrink);
      if (this.currentDrink == 0) return;
      console.log("^^^^^^^", this.currentDrink, this.prepareDrink);
      if (this.currentDrink >= this.todayTarget) {
        this.currentDrink = this.todayTarget;
        this.drawWater();
      } else {
        if (oldDrink != this.currentDrink) {
          console.log("画水");
          this.drawWater();
        }
      }
      this.prepareDrink = "";
      if (this.isCompleteTask()) {
        this.completeDays++;
      }
      this.saveRecord();
    },
    // 今日目标达成
    completeToday() {
      if(+this.currentDrink == +this.todayTarget){
        this.currentDrink=0
      }
      
    },
    // 清空水杯
    clearCup(clearnCurrent = false) {
      const ctx = this.$refs.canvas.getContext("2d");
      // 清空画布
      ctx.clearRect(0, 0, 390, 650);
      // 重新绘制水杯
      this.drawCup();
      // 清空数据
      if (!clearnCurrent) {
        this.currentDrink = 0;
      }
      this.prepareDrink = "";
      this.saveRecord();
    },
    // 判断是否完成任务
    isCompleteTask() {
      return this.currentDrink == this.todayTarget;
    },
    // 处理小数点保留
    dealDecimasl(num) {
      let str = num.toString();
      let result;
      if (str.indexOf(".") != -1) {
        result = str.substring(0, str.indexOf(".") + 4);
      } else {
        result = str;
      }
      return +result;
    },
    // 数据本地存储
    saveRecord() {
      let todayTarget = this.todayTarget;
      let currentDrink = this.currentDrink;
      let completeDays = this.completeDays;
      let dataRecord = {
        todayTarget,
        currentDrink,
        completeDays,
      };
      localStorage.setItem("drinkWaterRecord", JSON.stringify(dataRecord));
    },
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
      <!-- <div class="todayTarget">已完成天数：{{ completeDays }} 天</div> -->
      <canvas width="380" height="650" ref="canvas"></canvas>
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
          <button @click="completeToday">完成目标</button>
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
#app {
  height: 100vh;
}
.title {
  font-size: 4rem;
  text-align: center;
  background: linear-gradient(
    103.3deg,
    rgb(252, 225, 208) 30%,
    rgb(255, 173, 214) 55.7%,
    rgb(162, 186, 245) 81.8%
  );
}
.cup {
  position: relative;
  width: 100%;
}
.todayTarget {
  font-size: 2.5rem;
  padding-top: 1rem;
  padding-left: 1rem;
}
.button {
  position: absolute;
  width: 100%;
  bottom: 7.5rem;
}
.btn1 {
  display: flex;
  width: 100%;
  padding-left: 2rem;
  margin-bottom: 1rem;
}
input {
  border-color: rgb(61, 61, 237);
  padding-left: 1rem;
  border-radius: 1rem;
}
button {
  width: 10rem;
  height: 3.5rem;
  border-radius: 1rem;
  border: none;
  background-image: linear-gradient(
    110.6deg,
    rgb(179, 157, 219) 7%,
    rgb(150, 159, 222) 47.7%,
    rgb(24, 255, 255) 100.6%
  );
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
.btn2 button {
  margin-bottom: 1rem;
}
</style>
