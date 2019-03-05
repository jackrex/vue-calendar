<template>
  <div class="dialog" v-show="showMask">
    <div class="dialog-container">
      <div class="title-container">
        <button class="close" @click="closeMask"></button>
        <div class="dialog-title">{{title}}</div>
      </div>
      <div class="weeks">
        <div class="week" v-for="(wk,key) in weekName" :key="key">{{wk}}</div>
      </div>

      <div class="main">
        <div class="day" v-for="(day,key) in dateArr" :key="key" @click="clickDate(day,key)">
          <div
            class="value"
            :class="(day.flag && 'disbaled') || (day.isActive && 'on')"
          >{{day.value}}</div>
          <div class="parameter"></div>
        </div>
      </div>

      <div class="btns">
        <div v-if="type != 'confirm'" class="default-btn" @click="closeBtn">{{cancelText}}</div>
        <div v-if="type == 'danger'" class="danger-btn" @click="dangerBtn">{{dangerText}}</div>
        <div v-if="type == 'confirm'" class="confirm-btn" @click="confirmBtn">{{confirmText}}</div>
      </div>
      <div class="close-btn" @click="closeMask">
        <i class="iconfont icon-close"></i>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    value: {},
    // 类型包括 defalut 默认， danger 危险， confirm 确认，
    type: {
      type: String,
      default: "default"
    },
    parentTime: {
      // 当前时间，可由父级传入指定时间
      type: String,
      default: "",
      required: false
    },
    content: {
      type: String,
      default: ""
    },
    title: {
      type: String,
      default: ""
    },
    cancelText: {
      type: String,
      default: "取消"
    }
  },
  data() {
    let today = new Date();
    return {
      today: today,
      weekName: ["日", "一", "二", "三", "四", "五", "六"],
      showMask: false,
      currentDate: "", // 当前时间
      currentYear: "", // 当年
      currentMonth: "", // 当月
      currentDay: "",
      tempDay: "", // 切换时用到的临时变量
      firstDay: "", // 当月第一天是星期几
      monthDays: [], // 12个月份中每个月的天数
      showTime: "", // 用于显示的年月份
      dateArr: [] // 用于循环的日
    };
  },
  methods: {
    init(time) {
      // 初始化
      this.currentDate = time;
      this.currentYear = time.getFullYear();
      this.currentMonth = time.getMonth();
      this.currentDay = this.tempDay || time.getDate();
      this.getMonthDays();
      this.getFirstDay();
      this.calcullateDates();
      this.showTime =
        this.currentYear +
        "/" +
        (this.currentMonth + 1) +
        "/" +
        this.currentDay;
      this.tempDay = "";
    },
    isLeap(year) {
      // 计算闰年
      return year % 100 == 0
        ? year % 400 == 0
          ? 1
          : 0
        : year % 4 == 0
        ? 1
        : 0;
    },
    getMonthDays() {
      // 预先设置每月天数并计算二月天数
      this.monthDays = new Array(
        31,
        28 + this.isLeap(this.currentYear),
        31,
        30,
        31,
        30,
        31,
        31,
        30,
        31,
        30,
        31
      );
    },
    getFirstDay() {
      // 获取当月第一天是星期几
      this.firstDay = new Date(this.currentYear, this.currentMonth, 1).getDay();
    },
    calcullateDates() {
      // 计算月内天数，共显示5行，所以循环35次，超出当月天数的要重新计算
      let tempArr = [];
      for (let i = 0; i < 35; i++) {
        let dateValue = i - this.firstDay + 1; // 当月
        let clickFlag = false; // 是否可点击
        let isActive = false; // 是否选中
        let time = {
          // 用于传回父级的时间
          year: this.currentYear,
          month: this.currentMonth + 1,
          day: dateValue
        };
        if (dateValue <= 0 && this.currentMonth != 0) {
          // 上月
          dateValue = this.monthDays[this.currentMonth - 1] + dateValue;
          clickFlag = true;
          time.month = this.currentMonth;
          time.day = dateValue;
        } else if (dateValue <= 0 && this.currentMonth == 0) {
          // 去年
          dateValue = this.monthDays[this.monthDays.length - 1] + dateValue;
          clickFlag = true;
          time.year = this.currentYear - 1;
          time.month = this.currentMonth + 1;
          time.day = dateValue;
        } else if (dateValue > this.monthDays[this.currentMonth]) {
          // 下月
          dateValue = dateValue - this.monthDays[this.currentMonth];
          clickFlag = true;
          time.month = this.currentMonth + 1 + 1;
          time.day = dateValue;
        } else if (
          (this.currentDay < dateValue &&
            this.today.getMonth() <= this.currentMonth) ||
          this.today.getMonth() < this.currentMonth ||
          this.today.getFullYear() < this.currentYear
        ) {
          // 大于今天日期不能点击
          clickFlag = true;
        }
        if (dateValue == this.currentDay) {
          isActive = true;
        }
        tempArr.push({
          value: dateValue,
          flag: clickFlag,
          isActive,
          dateObj: time
        });
      }
      this.dateArr = tempArr;
    },
    nextMonth() {
      // 下月
      let time = new Date(this.currentYear, this.currentMonth + 2, 0);
      this.tempDay = this.currentDay;
      this.init(time);
    },
    preMonth() {
      // 上月
      let time = new Date(this.currentYear, this.currentMonth, 0);
      this.tempDay = this.currentDay;
      this.init(time);
    },
    clickDate(day, key) {
      // 点击事件，向父组件传递所点击的年月日
      if (day.flag) {
        return false;
      }
      // console.log(day,key);
      this.showTime =
        day.dateObj.year + "/" + day.dateObj.month + "/" + day.dateObj.day;
      this.dateArr.forEach((item, index) => {
        this.dateArr[index].isActive = index == key;
      });
      this.$emit("getQueryByDate", this.showTime);
    },
    closeMask() {
      this.showMask = false;
    },
    closeBtn() {
      this.$emit("cancel");
      this.closeMask();
    }
  },
  mounted() {
    this.showMask = this.value;
    let time = this.parentTime ? new Date(this.parentTime) : new Date(); // 如果父级传入就以父级为准，反之默认为当前时间
    this.init(time); // 初始化数据
    this.$emit("getQueryByDate", this.showTime);
  },
  watch: {
    value(newVal, oldVal) {
      this.showMask = newVal;
      newVal = oldVal;
    },
    showMask(val) {
      this.$emit("input", val);
    }
  }
};
</script>

<style lang="less" scoped>
.dialog {
  position: fixed;
  top: 0;
  bottom: 0;
  left: 0;
  right: 0;
  background: rgba(0, 0, 0, 0.6);
  z-index: 9999;
  .dialog-container {
    width: 900px;
    height: 680px;
    background: #ffffff;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    border-radius: 8px;
    position: relative;

    .title-container {
      display: flex;

      position: relative;
      .close {
        margin-top: 10px;
        margin-left: 20px;
        width: 32px;
        height: 32px;
        background-image: url(http://lc-heisafnh.cn-n1.lcfile.com/564f3d6a9950566de577/search.png);
      }
      .dialog-title {
        width: 100%;
        height: 60px;
        font-size: 18px;
        color: #696969;
        font-weight: 600;
        padding: 16px 50px 0 20px;
        box-sizing: border-box;
      }
    }

    .weeks .week,
    .main .day {
      width: 2.28rem;
      height: 1.4rem;
      line-height: 1.2rem;
      text-align: center;
      display: inline-block;
    }

    .main {
      width: 100%;
      /* height:100%; */
    }
    .main .day div {
      display: block;
    }
    .main .day .value {
      height: 1rem;
      width: 1rem;
      margin: 0 auto;
      border-radius: 50%;
      color: #fff;
      line-height: 1rem;
    }
    .main .day .value.on {
      background-color: #fff;
      color: #00a4ff;
    }
    .main .day .value.disbaled {
      color: #ccc;
    }
    .main .day .parameter {
      width: 0.2rem;
      height: 0.2rem;
      border-radius: 50%;
      background-color: #fff;
      margin: 0.05rem auto;
    }

    .content {
      color: #797979;
      line-height: 26px;
      padding: 0 20px;
      box-sizing: border-box;
    }
    .inp {
      margin: 10px 0 0 20px;
      width: 200px;
      height: 40px;
      padding-left: 4px;
      border-radius: 4px;
      border: none;
      background: #efefef;
      outline: none;
      &:focus {
        border: 1px solid #509ee3;
      }
    }
    .btns {
      width: 100%;
      height: 60px;
      // line-height: 60px;
      position: absolute;
      bottom: 0;
      left: 0;
      text-align: right;
      padding: 0 16px;
      box-sizing: border-box;
      & > div {
        display: inline-block;
        height: 40px;
        line-height: 40px;
        padding: 0 14px;
        color: #ffffff;
        background: #f1f1f1;
        border-radius: 8px;
        margin-right: 12px;
        cursor: pointer;
      }
      .default-btn {
        color: #787878;
        &:hover {
          color: #509ee3;
        }
      }
      .danger-btn {
        background: #ef8c8c;
        &:hover {
          background: rgb(224, 135, 135);
        }
        &:active {
          background: #ef8c8c;
        }
      }
      .confirm-btn {
        color: #ffffff;
        background: #509ee3;
        &:hover {
          background: #6fb0eb;
        }
      }
    }
    .close-btn {
      position: absolute;
      top: 16px;
      right: 16px;
      width: 30px;
      height: 30px;
      line-height: 30px;
      text-align: center;
      font-size: 18px;
      cursor: pointer;
      &:hover {
        font-weight: 600;
      }
    }
  }
}
</style>