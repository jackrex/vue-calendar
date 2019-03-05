<template>
  <div class="dialog" v-show="showMask">
    <div class="dialog-container">
      <div class="title-container">
        <button class="close" @click="closeMask"></button>
        <div class="dialog-title">{{title}}</div>
      </div>

      <template>
        <div id="calendar">
          <div class="date-title">
            <div>日</div>
            <div>一</div>
            <div>二</div>
            <div>三</div>
            <div>四</div>
            <div>五</div>
            <div>六</div>
          </div>
          <div class="calendar-container" v-on:scroll="scrollDebounce">
            <div class="date-container premonth">
              <div class="date-header">{{ `${preDivCalendar.year}年${preDivCalendar.month + 1}月` }}</div>
              <div class="date-board">
                <div
                  v-for="i in new Date(preDivCalendar.year, preDivCalendar.month, 1).getDay()"
                  :key="-i"
                ></div>
                <div
                  class="date-block"
                  v-for="i in getMonthDayByMonthYear(preDivCalendar.month, preDivCalendar.year)"
                  :key="i"
                >
                <span >{{ i }}</span>
                </div>
              </div>
            </div>

            <div class="date-container middlemonth">
              <div
                class="date-header"
              >{{ `${middleDivCalendar.year}年${middleDivCalendar.month + 1}月` }}</div>
              <div class="date-board">
                <div
                  v-for="i in new Date(middleDivCalendar.year, middleDivCalendar.month, 1).getDay()"
                  :key="-i"
                ></div>
                <div
                  class="date-block"
                  :class="{todayBlock: i === currentSelectCalendar.day & middleDivCalendar.month === currentSelectCalendar.month & middleDivCalendar.year === currentSelectCalendar.year}"
                  v-for="i in getMonthDayByMonthYear (middleDivCalendar.month, middleDivCalendar.year)"
                  :key="i"
                  @click.stop="dateClickHandle(i, middleDivCalendar.month)"
                >
                  <span v-if="i === new Date().getDate() & middleDivCalendar.month === new Date().getMonth()">今天</span>
                  <span v-else-if="i === new Date().getDate() + 1 & middleDivCalendar.month === new Date().getMonth()">明天</span>
                  <span v-else-if="i < new Date().getDate() + 1 & middleDivCalendar.month === new Date().getMonth()"></span>
                  <span v-else-if="i < new Date().getDate() + 1 & middleDivCalendar.month < new Date().getMonth() & middleDivCalendar.year < new Date().getFullYear()"></span>
                  <span v-else>{{ i }}</span>

                </div>
              </div>
            </div>
            <div class="date-container nextmonth">
              <div
                class="date-header"
              >{{ `${nextMiddleDivCalendar.year}年${nextMiddleDivCalendar.month + 1}月` }}</div>
              <div class="date-board">
                <div
                  v-for="i in new Date(nextMiddleDivCalendar.year, nextMiddleDivCalendar.month, 1).getDay()"
                  :key="-i"
                ></div>
                <div
                  class="date-block"
                  v-for="i in getMonthDayByMonthYear (nextMiddleDivCalendar.month, nextMiddleDivCalendar.year)"
                  :key="i"
                >
                  <span>{{ i }}</span>
                </div>
              </div>
            </div>
          </div>
        </div>
      </template>
    </div>
  </div>
</template>



<script>
import monthUtil from "./monthUtil.js";

export default {
  name: "calendarSlider",
  props: {
    value: {},
    // 类型包括 defalut 默认， danger 危险， confirm 确认，
    type: {
      type: String,
      default: "default"
    },
    events: {
      type: Array,
      default: function() {
        return [];
      }
    }
  },
  data() {
    return {
      showMask: false,
      currentSelectCalendar: {year: 2019, month: 2, day: 14},
      scrollTimer: null, // 滚动定时器，用于滚动事件防抖动
      preDivCalendar: { year: 2019, month: 2 }, // 第一个div显示的日历
      middleDivCalendar: { year: 2019, month: 3 }, // 中间div显示的日历
      nextMiddleDivCalendar: { year: 2019, month: 4 } // 最后一个div显示的日历
    };
  },

  methods: {
    closeMask() {
      this.showMask = false;
    },
    closeBtn() {
      this.$emit("cancel");
      this.closeMask();
    },
    dateClickHandle (day, month) {
        console.log(day, month);
        this.currentSelectCalendar.month = month;
        this.currentSelectCalendar.day = day;
        this.$emit('dateClick', date)
    },
    getPrevMonth: function(m, y) {
      // 获取上一个月
      let month = m || 11;
      let year = y;
      year -= m === 0;
      month -= m !== 0;
      return { year, month };
    },
    getNextMonth: function(m, y) {
      // 获取下一个月
      let month = m;
      let year = y;
      year += m === 11;
      month = (1 + month) % 12;
      return { year, month };
    },
    getMonthDayByMonthYear(month, year) {
      // 获取某年某月的天数
      // 判断是否为闰年
      const isLeap =
        year % 100 === 0 ? (year % 400 === 0 ? 1 : 0) : year % 4 === 0 ? 1 : 0;
      const monthDay = [
        31,
        28 + isLeap,
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
      ]; // 数组中的每一项代表每个月的天数
      return monthDay[month];
    },
    scrollDebounce(e) {
      clearTimeout(this.scrollTimer);
      let self = this;
      // 设置定时器，防止scroll抖动
      this.scrollTimer = setTimeout(function() {
        if (e.target.scrollTop === 0) {
          self.middleDivCalendar = self.getPrevMonth(
            self.middleDivCalendar.month,
            self.middleDivCalendar.year
          );
          self.$nextTick(function() {
            // DOM 更新了
            const premonth = document.querySelector(".premonth");
            e.target.scrollTop = premonth.offsetHeight;
          });
        }
        if (
          e.target.scrollTop + e.target.clientHeight + 1 >=
          e.target.scrollHeight
        ) {
          self.middleDivCalendar = self.getNextMonth(
            self.middleDivCalendar.month,
            self.middleDivCalendar.year
          );
          self.$nextTick(function() {
            // DOM 更新了
            const nextmonth = document.querySelector(".nextmonth");
            e.target.scrollTop =
              e.target.scrollHeight -
              e.target.clientHeight -
              nextmonth.offsetHeight;
          });
        }
      }, 100);
    }
  },
  //初始化
  mounted() {
    this.showMask = this.value;
    this.middleDivCalendar = {
      year: new Date().getFullYear(),
      month: new Date().getMonth()
    };

    this.currentSelectCalendar = {
      year: new Date().getFullYear(),
      month: new Date().getMonth(),
      day: new Date().getDate()
    };

    this.$nextTick(function() {
      // DOM更新完成后
      const today = document.querySelector(".todayBlock");
      today.scrollIntoView();
    });
  },

  watch: {
    value(newVal, oldVal) {
      this.showMask = newVal;
      newVal = oldVal;
    },
    showMask(val) {
      this.$emit("input", val);
    },
    isShow() {},
    middleDivCalendar() {
      // 监听中间div日历的时间,根据中间div的日历获取上下div的日历
      this.preDivCalendar = this.getPrevMonth(
        this.middleDivCalendar.month,
        this.middleDivCalendar.year
      );
      this.nextMiddleDivCalendar = this.getNextMonth(
        this.middleDivCalendar.month,
        this.middleDivCalendar.year
      );
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
    display: flex;
    flex-direction: column;

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
  }
}

.date-title {
  display: flex;
  padding: 0.1rem 0;
  text-align: center;
  border-bottom: 0.01rem solid #d1d1d1;
  div {
    flex: 1;
  }
}
.calendar-container {
  overflow-y: scroll;
  height: 300px;
  overflow-scrolling: touch;
  transition: height 0.5s ease;
  .date-container {
    .date-header {
      text-align: center;
      padding: 0.07rem 0;
      background-color: #f8f8f8;
    }
    .date-block {
      background-color: white;
    }

    .date-board {
      display: flex;
      display: -webkit-flex;
      flex-wrap: wrap;
      -webkit-flex-wrap: wrap;
      text-align: center;
      div {
        width: 14.28571%;
        padding: 0.1rem 0;
      }
      .todayBlock {
        background-color: #fd992e;

        span {
          color: white;
        }
      }
    }
  }
}
</style>