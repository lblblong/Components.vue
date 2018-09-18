<template>
  <div id="calendar">
    <header class="header" :style="toolbarSize">
      <span @click="toMonth('last')">◀</span>
      {{currentMonth.format('YYYY 年 M 月')}}
      <span @click="toMonth('next')">▶</span>
    </header>
    <section class="week-title" :style="toolbarSize">
      <span class="week">一</span>
      <span class="week">二</span>
      <span class="week">三</span>
      <span class="week">四</span>
      <span class="week">五</span>
      <span class="week">六</span>
      <span class="week">日</span>
    </section>
    <section class="days" :style="daysSize">
      <div @click="select(day)" class="day" :class="[{today:day.format('YYYY-M-D')==dayjs().format('YYYY-M-D')},{active:day.format('YYYY-M-D')==value}]" v-for="day in days" :key="day.format('YYYY-M-D')">
        <div :style="{visibility:day.dot?'visible':'hidden'}" class="red-dot"></div>
        <div v-if="!day.currentMonth" class="content" style="color:#aaa;">{{day.date()}}</div>
        <div v-else class="content">{{day.date()}}</div>
        <div class="msg" v-html="day.message" style="font-size:0.01rem"></div>
      </div>
    </section>
  </div>
</template>

<script>
import dayjs from 'dayjs'
export default {
  props: {
    /**
     * 选中日期
     */
    value: {
      default: dayjs().format('YYYY-M-D')
    },
    // 标点日期列表
    dotdays: {
      type: Array,
      default: () => {
        return []
      }
    },
    /**
     * 展示消息日期列表
     * 格式：{date:'2016-7-5',message:'未读'}
     */
    msgdays: {
      type: Array,
      default: () => {
        return []
      }
    }
  },
  watch: {
    dotdays(newValue, oldValue) {
      this.reDays()
    },
    msgdays(newValue, oldValue) {
      this.reDays()
    }
  },
  data() {
    return {
      toolbarSize: {
        width: '',
        height: ''
      },
      daysSize: {
        width: '',
        height: ''
      },
      days: [],
      currentMonth: dayjs(this.value).startOf('month')
    }
  },
  created() {},
  mounted() {
    this.dayjs = dayjs
    this.measure()
    this.reDays(this.value)
  },
  methods: {
    measure() {
      let $parent = this.$parent
      while ($parent.$el.offsetWidth == 0) {
        $parent = $parent.$parent
      }
      let parentWidth = $parent.$el.offsetWidth
      if (parentWidth < 320) {
        // 小于 320 宽高固定
        this.toolbarSize.width = 320 + 'px'
        this.toolbarSize.height = 320 / 7 / 1.5 + 'px'
        this.daysSize.width = 320 + 'px'
        this.daysSize.height = 320 / 7 * 5 + 'px'
        // } else if (parentWidth > 960) {
        //     // 大于 960 宽高固定
        //     this.toolbarSize.width = 960 + 'px'
        //     this.toolbarSize.height = 960 / 7 / 1.5 + 'px'
        //     this.daysSize.width = 960 + 'px'
        //     this.daysSize.height = 960 / 7 * 5 + 'px'
      } else {
        // 宽高自适应
        this.toolbarSize.width = parentWidth + 'px'
        this.toolbarSize.height = parentWidth / 7 / 1.5 + 'px'
        this.daysSize.width = parentWidth + 'px'
        this.daysSize.height = parentWidth / 7 * 5 + 'px'
      }
    },
    reDays(date) {
      this.days = []
      let currentDay = dayjs(date)
      let startDay = currentDay.startOf('month')
      let endDay = currentDay.endOf('month')
      // 添加上个月最后的几天
      let lastMonth = []
      let week = startDay.day() == 0 ? 7 : startDay.day()
      for (let i = 1; i < week; i++) {
        let day = startDay.subtract(i, 'day')
        day.currentDay = false
        lastMonth.push(day)
      }
      this.days.push(...lastMonth.reverse())
      // 添加目标月
      for (let i = 0; i < startDay.daysInMonth(); i++) {
        let day = startDay.add(i, 'day')
        day.currentMonth = true
        this.days.push(day)
      }
      // 添加下个月的头几天
      let nextMonth = []
      week = endDay.day() == 0 ? 7 : endDay.day()
      for (let i = 1; i <= 7 - week; i++) {
        let day = endDay.add(i, 'day')
        day.currentDay = false
        nextMonth.push(day)
      }
      this.days.push(...nextMonth)
      // 如果多出 35 天需截断
      this.days = this.days.slice(0, 35)
      // 标点 与 message
      this.days.forEach(day => {
        day.dot = false
        day.message = '&nbsp;'
        let strDay = day.format('YYYY-M-D')
        for (let i = 0; i < this.dotdays.length; i++) {
          let strDotDay = dayjs(this.dotdays[i]).format('YYYY-M-D')
          if (strDay == strDotDay) day.dot = true
        }
        for (let i = 0; i < this.msgdays.length; i++) {
          let strMsgDay = dayjs(this.msgdays[i].date).format('YYYY-M-D')
          if (strDay == strMsgDay) day.message = this.msgdays[i].message
        }
      })
    },
    toMonth(action) {
      let tmpCurrent = this.currentMonth
      if (action == 'next') {
        this.currentMonth = tmpCurrent.add(1, 'month')
      } else if (action == 'last') {
        this.currentMonth = tmpCurrent.subtract(1, 'month')
      }
      this.reDays(this.currentMonth.format('YYYY-M-D'))
      // 触发月份改变事件
      this.$emit('changeMonth', this.currentMonth.format('YYYY-M-D'))
      // 触发年份改变事件
      if (tmpCurrent.format('YYYY') != this.currentMonth.format('YYYY')) {
        this.$emit('changeYear', this.currentMonth.format('YYYY-M-D'))
      }
    },
    select(day) {
      this.$emit('input', day.format('YYYY-M-D'))
      this.$emit('change', day.format('YYYY-M-D'))
      this.reDays(day.format('YYYY-M-D'))
    }
  }
}
</script>

<style lang="less" scoped>
#calendar {
  width: 100%;
  background-color: white;
  .header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 0 50px;
    background-color: #37bc98;
    color: #f5f7fa;
  }
  .week-title {
    display: grid;
    grid-template-columns: auto auto auto auto auto auto auto;
    background-color: #37bc98;
    .week {
      display: flex;
      justify-content: center;
      align-items: center;
      color: #f5f7fa;
    }
  }
  .days {
    display: grid;
    grid-template-columns: auto auto auto auto auto auto auto;
    grid-template-rows: 20% 20% 20% 20% 20%;
    .day {
      display: flex;
      flex-direction: column;
      justify-content: center;
      align-items: center;
      box-sizing: border-box;
      .red-dot {
        width: 0;
        height: 0;
        border: 3px solid #ccd1d9;
        border-radius: 50%;
      }
      .msg {
        color: #48cfad;
      }
      .content {
        font-weight: bold;
      }
    }

    .today {
      .content {
        color: #37bc98;
      }
    }

    .active {
      .content {
        color: #ed5565;
      }
    }
  }
}
</style>