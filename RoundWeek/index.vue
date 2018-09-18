<template>
  <div id="round-week">
    <div @click="change(item)" class="item" v-for="item in roundWeek" :key="item.day()" :class="{mark:isMark(item)}">
      <div class="dayEn">{{item.dayEn()}}</div>
      <div class="dayNum">{{item.date()}}</div>
    </div>
  </div>
</template>

<script>
import dayjs from 'dayjs'
export default {
  props: {
    /**
     * 目标时间
     */
    value: {
      default: dayjs().format('YYYY-M-D')
    },
    /**
     * 标记日期(特殊日期)
     */
    marks: {
      default: () => {
        return []
      }
    }
  },
  computed: {
    roundWeek: function() {
      let center = dayjs(this.value)
      let left1 = center.subtract(1, 'day')
      let left2 = center.subtract(2, 'day')
      let left3 = center.subtract(3, 'day')
      let right1 = center.add(1, 'day')
      let right2 = center.add(2, 'day')
      let right3 = center.add(3, 'day')
      let rw = [left3, left2, left1, center, right1, right2, right3]
      rw.forEach(day => {
        day.dayEn = (num => {
          return function() {
            switch (num) {
              case 1:
                return '周一'
              case 2:
                return '周二'
              case 3:
                return '周三'
              case 4:
                return '周四'
              case 5:
                return '周五'
              case 6:
                return '周六'
              case 0:
                return '周日'
            }
          }
        })(day.day())
      })
      return rw
    }
  },
  watch: {
    value(newValue, oldValue) {
      this.$emit('change', newValue)
    }
  },
  methods: {
    change(item) {
      this.$emit('input', item.format('YYYY-M-D'))
    },
    isMark(date) {
      for (let i = 0; i < this.marks.length; i++) {
        let day = dayjs(this.marks[i]).format('YYYY-M-D')
        if (dayjs(day).isSame(date)) {
          return true
        }
      }
      return false
    }
  }
}
</script>

<style lang="less" scoped>
#round-week {
  display: flex;
  height: 1.1rem;
  background-color: white;
  .item {
    flex: 1;
    display: flex;
    color: #a9a9a9;
    flex-direction: column;
    align-items: center;
    .dayEn {
      font-size: 14px;
      height: 0.28rem;
    }
    .dayNum {
      width: 0.56rem;
      height: 0.56rem;
      margin-top: 0.12rem;
      display: flex;
      justify-content: center;
      align-items: center;
    }
    &:nth-child(4) {
      color: black;
      .dayNum {
        padding: 0.1rem;
        border: 2px solid #3e96fe;
        border-radius: 50%;
        box-shadow: 0px 0.08rem 0.1rem 0.04rem #e3ebff;
        color: #3e96fe;
      }
      .dayEn {
        font-weight: bold;
      }
    }
  }
  .mark {
    .dayNum {
      color: #ed5565;
      font-weight: bold;
    }
  }
}
</style>