<template>
  <div class="container" ref="container">
    <transition>
      <div class="modal-wrap" :style="modalStyle">
        <div class="content-wrap">
          <span class="day">{{dayFormat2(modalDay.day)}}</span>
          <span v-for="plan in modalDay.longPlans"
          :key="plan.index"
          class="plan -long -blue">
            {{ plan.title }}
          </span>
          <span v-for="plan in modalDay.plans" class="plan -short" :key="plan.index" :class="colorClass(plan.index)">
            <span class="time">{{plan.start_time}}</span>{{plan.name}}
          </span>
          <span class="back-btn" @click="close"></span>
        </div>
      </div>
    </transition>
    <div class="WebCalender">
      <div class="year-month-wrap">
        <ul class="month-btn-list">
          <li class="item back" @click="goPrevMonth"></li>
          <li class="item next" @click="goNextMonth"></li>
        </ul>
        <div class="txt-wrap">
          <span class="item year">{{year}}</span>
          <span class="item month">{{month}}</span>
        </div>
      </div>
      <div class="calender-wrap">
        <div class="weekly-wrap">
          <div class="item sun">
            <span>日</span>
          </div>
          <div class="item">
            <span>月</span>
          </div>
          <div class="item">
            <span>火</span>
          </div>
          <div class="item">
            <span>水</span>
          </div>
          <div class="item">
            <span>木</span>
          </div>
          <div class="item">
            <span>金</span>
          </div>
          <div class="item sat">
            <span>土</span>
          </div>
        </div>
        <div class="days-wrap">
          <div v-for="day in calendarData" :key="day.index" class="item" :class="{'-mult': isMulti(day)}" :ref="`id_${day}`" @click="detailPlan($event, day); close(day)">
            <div class="content-wrap">
              <span class="day" :class="{'-hide':isHide(day)}">{{dayFormat(day)}}</span>
              <div class="plan-wrap">
                <span v-for="plan in longPlans[day]"
                :key="plan.index"
                class="plan -long -blue"
                :class="longPlanStyleClass(plan, day)">
                {{ plan.title }}
                </span>
                <span v-for="(plan, index) in dayPlans[day]"
                :key="index"
                class="plan -short"
                :class="shortPlanStyleClass(index, day)">
                  <span class="time">{{plan.start_time}}</span>{{plan.name}}
                </span>
              </div>
              <span v-if="duplicateCount[day] > 0" class="mult-num">{{duplicateCount[day]}}</span>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import moment from 'moment'
import _ from 'lodash'
export default {
  data () {
    return {
      selectedDate: null, // 今選択している日付,
      current: 0,
      long_plans: [],
      day_plans: [],
      color: ['-red', '-orange', '-yellow', '-light-green', '-green', '-sky-blue', '-blue', '-purple'],
      modalDay: {
        day: '',
        title: '',
        plans: [],
        longPlans: []
      },
      duplicateCount: {}, // どの日付に何個のプランが重複しているか
      planIndex: {},
      modalStyle: {
        // display: 'none',
        top: '0px',
        left: '0px',
        'z-index': -10,
        transition: 'z-index 0.5s, height 1s, width 1s,opacity 1s ',
        // transition: '.3s',
        width: '146px',
        height: '126px',
        opacity: 0
      },
      modalMiniWidth: 146,
      modalMiniHeight: 126,
      modalMaxWidth: 245,
      modalMaxHeight: 300,
      transitionOn: 'z-index 0.5s, height 1s, width 1s,opacity 1s',
      show: false
    }
  },
  methods: {
    /**
     * 来月のカレンダーを表示する
     */
    goNextMonth () {
      this.close()
      this.current++
    },
    /**
     * 先月のカレンダーを表示する
     */
    goPrevMonth () {
      this.close()
      this.current--
    },
    /**
     * カラー用のクラスを返す
     */
    colorClass (number) {
      return this.color[number % 8]
    },
    /**
     * YYYYMMDD形式の日付を受け取って日付のみ返す
     */
    dayFormat (date) {
      return moment(date, 'YYYYMMDD').format('DD')
    },
    /**
     * その週のどこまで長期講習を表示させるか計算してクラス名を返す
     */
    longPlanStatus (longPlan, date) {
      // 指定された日付から何日間あるか
      const dateDiff = this.dateDifference(date, longPlan.end_date) + 1
      // 来週に跨いでいた場合
      if (7 - this.weekDay(date) < dateDiff) {
        return `-l${7 - this.weekDay(date)}`
      }
      return `-l${dateDiff}`
    },
    /**
     * 曜日の番号を返す 日から順に0,1...
     */
    weekDay (date) {
      return moment(date, 'YYYYMMDD').day()
    },
    /**
     * 週の始まりかどうかを返す
     */
    isWeekStart (date) {
      return this.weekDay(date) === 0
    },
    dateDifference (date1, date2) {
      const diffInDays = moment(date1, 'YYYYMMDD').diff(moment(date2, 'YYYYMMDD'), 'days')
      return Math.abs(diffInDays)
    },
    shortPlanStyleClass (planIndex, date) {
      let style = this.colorClass(planIndex)
      // 期間重複数
      let duplicateCount = this.duplicateCount[date]
      let margin = 1
      for (let i = 1; i <= duplicateCount + 1; i++) {
        // planIndexに存在するIndexは使わない
        if (this.planIndex[date].find(item => item === i)) {
          continue
        }
        margin = i
        this.fillPlanIndex(date, 1, i)
        break
      }
      return `${style} -top${margin}`
    },
    /**
     * 長期講習に付与するclassをまとめるメソッド
     */
    longPlanStyleClass (longPlan, date) {
      let style = this.longPlanStatus(longPlan, date)
      // 期間重複数
      let duplicateCount = this.duplicateCount[date]
      let margin = 1
      for (let i = 1; i <= duplicateCount + 1; i++) {
        // planIndexに存在するIndexは使わない
        if (this.planIndex[date].find(item => item === i)) {
          continue
        }
        margin = i
        // 指定された日付から何日間あるか
        const dateDiff = this.dateDifference(date, longPlan.end_date) + 1
        let j = 0
        // 来週に跨いでいた場合
        if (7 - this.weekDay(date) < dateDiff) {
          j = 7 - this.weekDay(date)
        } else {
          j = dateDiff
        }
        this.fillPlanIndex(date, j, i)
        break
      }
      return `${style} -top${margin}`
    },
    /*
     * 長期期間の時に他の日のplanIndexを埋めていく
     * date 夏期講習の要素が入る日
     * num その週の夏期講習の入る期間
     * index その日に使っている連番の数値
     */
    fillPlanIndex (date, num, index) {
      for (let i = 0; i < num; i++) {
        this.planIndex[moment(date, 'YYYYMMDD').add(i, 'days').format('YYYYMMDD')].push(index)
      }
    },
    /**
     * computedに書けばいいのかメソッドに書けばいいのか迷い中
     */
    planCount () {
      this.planIndex = {}
      let planCount = {}
      Object.keys(this.longPlansAll).forEach(date => {
        planCount[date] = 0
        this.planIndex[date] = []
        this.longPlansAll[date].forEach(() => planCount[date]++)
      })
      Object.keys(this.dayPlans).forEach(date => {
        this.dayPlans[date].forEach(() => planCount[date]++)
      })
      this.duplicateCount = planCount
      this.loopDuplicateCount = planCount
    },
    async detailPlan (event, date) {
      if (this.duplicateCount[date] === 0) {
        return
      }
      await this.close(null, true)
      // クリックした親要素のobject
      let obj = this.$refs[`id_${date}`][0].getBoundingClientRect()
      let x = obj.x - (this.$refs.container.getBoundingClientRect().x + 1)
      let y = obj.y + document.documentElement.scrollTop
      this.$set(this.modalStyle, 'transition', this.transitionOn)
      this.$set(this.modalStyle, 'top', `${y}px`)
      this.$set(this.modalStyle, 'left', `${x}px`)
      this.$set(this.modalStyle, 'width', `${this.modalMaxWidth}px`)
      this.$set(this.modalStyle, 'height', `${this.modalMaxHeight}px`)
      this.$set(this.modalStyle, 'opacity', 1)
      this.$set(this.modalStyle, 'z-index', 200)

      this.$set(this.modalDay, 'day', date)
      this.$set(this.modalDay, 'plans', this.dayPlans[date])
      this.$set(this.modalDay, 'longPlans', this.longPlansAll[date])
      this.show = true
    },
    close (date = null, transitionOff = false) {
      if (date != null && date === this.modalDay.day) {
        return
      }
      if (transitionOff) {
        this.$delete(this.modalStyle, 'transition')
      }
      this.$set(this.modalStyle, 'width', `${this.modalMiniWidth}px`)
      this.$set(this.modalStyle, 'height', `${this.modalMiniHeight}px`)
      this.$set(this.modalStyle, 'z-index', -10)
      this.$set(this.modalStyle, 'opacity', 0)
    },
    isMulti (date) {
      return this.duplicateCount[date] > 0
    },
    dayFormat2 (date) {
      return moment(date, 'YYYYMMDD').format('YYYY年MM月DD日')
    },
    isHide (date) {
      return this.currentMoment.format('MM') !== moment(date, 'YYYYMMDD').format('MM')
    }
  },
  created () {
    this.long_plans = [
      {id: 1, start_date: '20191012', end_date: '20191024', title: '長期講習1'},
      {id: 2, start_date: '20191019', end_date: '20191027', title: 'hogehoge'}
    ]
    this.day_plans = [
      {id: 1, day: '20191022', start_time: '19:00', end_time: '21:00', name: '数学'}
    ]
    this.planCount()
  },
  beforeUpdate () {
    this.planCount()
  },
  watch: {
    current (val) {
      this.planCount()
    }
  },
  computed: {
    currentMoment () {
      return moment().add(this.current, 'months')
    },
    year () {
      return this.currentMoment.format('YYYY年')
    },
    month () {
      return this.currentMoment.format('MM月')
    },
    calendarData () {
      // この月に何日まであるかを算出
      const numOfMonth = this.currentMoment.endOf('month').date()
      // const numOfPrevMonth = this.currentMoment.clone().add(-1, 'months').endOf('month').date()
      // この月の1日〜最終日までの配列
      const daysOfMonth = [...Array(numOfMonth).keys()].map(i => ++i)
      // 1日の曜日（0~6の数値で取得）
      const firstWeekDay = this.currentMoment.startOf('month').weekday()
      // 週ごとの二次元配列を生成
      const data = [...Array(6)].map((empty, weekIndex) =>
        [...Array(7)].map((empty, dayIndex) => {
          const i = 7 * weekIndex + dayIndex - firstWeekDay
          if (i < 0) {
            return this.currentMoment.clone().date(1).add(i, 'days').format('YYYYMMDD')
          }
          if (i >= numOfMonth) {
            return this.currentMoment.clone().date(numOfMonth).add((i - numOfMonth + 1), 'days').format('YYYYMMDD')
          }
          return this.currentMoment.clone().date(daysOfMonth[i]).format('YYYYMMDD')
        })
      )
      const result = data.filter(week => week.filter(day => day <= this.currentMoment.clone().date(numOfMonth).format('YYYYMMDD')).length > 0)
      const list = result.reduce((pre, current) => { pre.push(...current); return pre }, [])
      return list
    },
    /**
     * 日ごとに(カレンダーの要素的に)存在する長期予定を返す
     */
    longPlans () {
      let longPlans = {}
      this.calendarData.map((date, index, array) => {
        const todayPlans = this.long_plans.filter(x => (x.start_date === date))
        const plansInTime = this.long_plans.filter(x => (x.start_date <= date && x.end_date >= date))
        const result = _.uniqBy([...todayPlans, ...plansInTime], 'id')
        if (!this.isWeekStart(date)) {
          longPlans[date] = todayPlans
          return
        }
        longPlans[date] = result
      })
      return longPlans
    },
    /**
     * 日ごとに(データ的に)存在する長期予定を返す
     */
    longPlansAll () {
      let longPlansAll = {}
      this.calendarData.map((date, index, array) => {
        const todayPlans = this.long_plans.filter(x => (x.start_date === date))
        const plansInTime = this.long_plans.filter(x => (x.start_date <= date && x.end_date >= date))
        const result = _.uniqBy([...todayPlans, ...plansInTime], 'id')
        longPlansAll[date] = result
      })
      return longPlansAll
    },
    /**
     * 日ごとのプランを計算
    */
    dayPlans () {
      let daysPlans = {}
      this.calendarData.map((date, index, array) => {
        const dayPlans = this.day_plans.filter(x => x.day === date)
        daysPlans[date] = dayPlans
      })
      return daysPlans
    }
  }
}
</script>
<style scoped>
*,
*:after,
*:before {
  -webkit-box-sizing: border-box;
  box-sizing: border-box;
  color: #333;
  font-family: 'Hiragino Kaku Gothic ProN', 'ヒラギノ角ゴ ProN W3', sans-serif
}

.container {
  position: relative;
  width: 1080px;
  height: 100vh;
  margin: 0 auto;
  border-left: solid 1px #ccc;
  border-right: solid 1px #ccc;
  padding: 64px calc((1080px - 986px) / 2) 0
}

.WebCalender {
  position: relative
}

.WebCalender .year-month-wrap {
  display: -webkit-box;
  display: -ms-flexbox;
  display: flex;
  margin-bottom: 12px
}

.WebCalender .year-month-wrap .month-btn-list {
  margin-right: 10px
}

.WebCalender .year-month-wrap .month-btn-list .item {
  background-color: #acacff;
  border-radius: 11px;
  display: inline;
  cursor: pointer;
  color: #fff;
  font-family: "Material Design Icons";
  font-size: 22px
}

.WebCalender .txt-wrap .item {
  font-size: 24px;
  font-family: "游ゴシック体", YuGothic, "游ゴシック", "Yu Gothic", "メイリオ", sans-serif
}

.WebCalender .calender-wrap {
  position: relative;
  background-color: #E2E2E2;
  border-radius: 5px;
  padding: 2px
}

.WebCalender .calender-wrap .weekly-wrap {
  padding: 0;
  width: 100%;
  display: -webkit-box;
  display: -ms-flexbox;
  display: flex
}

.WebCalender .calender-wrap .weekly-wrap .item {
  margin: 2px
}

.WebCalender .calender-wrap .weekly-wrap .item span {
  border-radius: 5px;
  background-color: #fff;
  color: #666;
  display: block;
  font-size: 12px;
  padding: 5px 0;
  text-align: center;
  width: 136px
}

.WebCalender .calender-wrap .weekly-wrap .sun span {
  background-color: #EA7979;
  color: #fff
}

.WebCalender .calender-wrap .weekly-wrap .sat span {
  background-color: #7979FF;
  color: #fff
}

.WebCalender .calender-wrap .days-wrap {
  display: -webkit-box;
  display: -ms-flexbox;
  display: flex;
  -ms-flex-wrap: wrap;
  flex-wrap: wrap
}

.WebCalender .calender-wrap .days-wrap .item {
  position: relative;
  margin: 2px;
  width: 136px;
  height: 116px
}

.WebCalender .calender-wrap .days-wrap .item:before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  height: 100%;
  width: 100%;
  z-index: 50
}

.WebCalender .calender-wrap .days-wrap .item .content-wrap {
  background-color: #fff;
  border-radius: 5px;
  width: 136px;
  height: 116px;
  padding: 6px 2px;
  -webkit-transition: .3s;
  transition: .3s
}

.WebCalender .calender-wrap .days-wrap .item .day {
  display: block;
  color: #666;
  font-size: 12px;
  font-family: "Roboto Condensed", sans-serif;
  text-align: center;
  line-height: 16px;
  margin: 0 auto 3px;
  width: 16px;
  z-index: 10
}

.WebCalender .calender-wrap .days-wrap .item .-hide {
  color: #aaa
}

.WebCalender .calender-wrap .days-wrap .item .plan-wrap {
  position: relative;
  height: 100%
}

.WebCalender .calender-wrap .days-wrap .item .plan {
  position: absolute;
  top: 0;
  font-size: 12px;
  margin-bottom: 2px;
  height: calc(1em + 6px)
}

.WebCalender .calender-wrap .days-wrap .item .plan.-short {
  display: block;
  width: 100%;
  overflow: hidden;
  padding: 3px 0;
  text-overflow: ellipsis;
  white-space: nowrap;
  display: block;
  height: calc(1em + 6px);
  padding: 3px 0
}

.WebCalender .calender-wrap .days-wrap .item .plan.-short:before {
  content: '';
  background-color: #EA7979;
  border-radius: 4px;
  padding: 2px 3px;
  margin-right: 4px;
  height: 8px;
  width: 8px
}

.WebCalender .calender-wrap .days-wrap .item .plan.-short.-red:before {
  background-color: #EA7979
}

.WebCalender .calender-wrap .days-wrap .item .plan.-short.-orange:before {
  background-color: #F8A748
}

.WebCalender .calender-wrap .days-wrap .item .plan.-short.-yellow:before {
  background-color: #F2E32D
}

.WebCalender .calender-wrap .days-wrap .item .plan.-short.-light-green:before {
  background-color: #88D64E
}

.WebCalender .calender-wrap .days-wrap .item .plan.-short.-green:before {
  background-color: #29C48F
}

.WebCalender .calender-wrap .days-wrap .item .plan.-short.-sky-blue:before {
  background-color: #6FD2F0
}

.WebCalender .calender-wrap .days-wrap .item .plan.-short.-blue:before {
  background-color: #7979FF
}

.WebCalender .calender-wrap .days-wrap .item .plan.-short.-purple:before {
  background-color: #BB76E6
}

.WebCalender .calender-wrap .days-wrap .item .plan.-short:before {
  content: '';
  background-color: #EA7979;
  border-radius: 4px;
  padding: 2px 3px;
  margin-right: 4px;
  height: 8px;
  width: 8px
}

.WebCalender .calender-wrap .days-wrap .item .plan.-short.-red:before {
  background-color: #EA7979
}

.WebCalender .calender-wrap .days-wrap .item .plan.-short.-orange:before {
  background-color: #F8A748
}

.WebCalender .calender-wrap .days-wrap .item .plan.-short.-yellow:before {
  background-color: #F2E32D
}

.WebCalender .calender-wrap .days-wrap .item .plan.-short.-light-green:before {
  background-color: #88D64E
}

.WebCalender .calender-wrap .days-wrap .item .plan.-short.-green:before {
  background-color: #29C48F
}

.WebCalender .calender-wrap .days-wrap .item .plan.-short.-sky-blue:before {
  background-color: #6FD2F0
}

.WebCalender .calender-wrap .days-wrap .item .plan.-short.-blue:before {
  background-color: #7979FF
}

.WebCalender .calender-wrap .days-wrap .item .plan.-short.-purple:before {
  background-color: #BB76E6
}

.WebCalender .calender-wrap .days-wrap .item .plan.-long {
  left: -2px;
  background-color: #EA7979;
  border-radius: 5px;
  color: #fff;
  padding: 3px 10px;
  width: 100%;
  z-index: 40
}

.WebCalender .calender-wrap .days-wrap .item .plan.-long.-red {
  background-color: #EA7979
}

.WebCalender .calender-wrap .days-wrap .item .plan.-long.-orange {
  background-color: #F8A748
}

.WebCalender .calender-wrap .days-wrap .item .plan.-long.-yellow {
  background-color: #F2E32D
}

.WebCalender .calender-wrap .days-wrap .item .plan.-long.-light-green {
  background-color: #88D64E
}

.WebCalender .calender-wrap .days-wrap .item .plan.-long.-green {
  background-color: #29C48F
}

.WebCalender .calender-wrap .days-wrap .item .plan.-long.-sky-blue {
  background-color: #6FD2F0
}

.WebCalender .calender-wrap .days-wrap .item .plan.-long.-blue {
  background-color: #7979FF
}

.WebCalender .calender-wrap .days-wrap .item .plan.-long.-purple {
  background-color: #BB76E6
}

.WebCalender .calender-wrap .days-wrap .item .plan.-long.-l1 {
  width: calc(140px * 1 - 4px)
}

.WebCalender .calender-wrap .days-wrap .item .plan.-long.-l2 {
  width: calc(140px * 2 - 4px)
}

.WebCalender .calender-wrap .days-wrap .item .plan.-long.-l3 {
  width: calc(140px * 3 - 4px)
}

.WebCalender .calender-wrap .days-wrap .item .plan.-long.-l4 {
  width: calc(140px * 4 - 4px)
}

.WebCalender .calender-wrap .days-wrap .item .plan.-long.-l5 {
  width: calc(140px * 5 - 4px)
}

.WebCalender .calender-wrap .days-wrap .item .plan.-long.-l6 {
  width: calc(140px * 6 - 4px)
}

.WebCalender .calender-wrap .days-wrap .item .plan.-long.-l7 {
  width: calc(140px * 7 - 4px)
}

.WebCalender .calender-wrap .days-wrap .item .plan.-top1 {
  top: 0
}

.WebCalender .calender-wrap .days-wrap .item .plan.-top2 {
  top: calc(1em + 8px)
}

.WebCalender .calender-wrap .days-wrap .item .plan.-top3 {
  top: calc(2 * (1em + 8px))
}

.WebCalender .calender-wrap .days-wrap .item .plan.-top4 {
  top: calc(3 * (1em + 8px))
}

.WebCalender .calender-wrap .days-wrap .item .time {
  font-family: "Roboto Condensed", sans-serif;
  font-weight: bold;
  color: #999;
  margin-right: 6px
}

.WebCalender .calender-wrap .days-wrap .item.-mult {
  position: relative
}

.WebCalender .calender-wrap .days-wrap .item.-mult:after {
  content: '';
  position: absolute;
  bottom: 0;
  background-color: #fff;
  opacity: .4;
  width: 100%;
  height: 30px;
  z-index: 20
}

.WebCalender .calender-wrap .days-wrap .item.-mult .mult-num {
  position: absolute;
  right: 0;
  bottom: 0;
  background-color: #fff;
  color: #999;
  font-family: "Roboto Condensed", sans-serif;
  font-size: 14px;
  padding: 8px 6px;
  z-index: 60
}

.WebCalender .calender-wrap .days-wrap .item.-mult .mult-num:before {
  content: '\f0f0';
  color: #aaa;
  font-family: "Material Design Icons";
  -webkit-transition: .3s;
  transition: .3s
}

.WebCalender .calender-wrap .days-wrap .item.-mult .mult-num:after {
  content: '';
  position: absolute;
  right: 0;
  bottom: 0;
  width: 0;
  height: 0;
  border-style: solid;
  border-width: 0 0 10px 10px;
  border-color: transparent transparent #999999 transparent
}

.WebCalender .calender-wrap .days-wrap .item.-mult:hover:after {
  height: 0
}

.WebCalender .calender-wrap .days-wrap .item.-mult:hover .content-wrap {
  position: absolute;
  cursor: pointer;
  top: 0;
  left: 0;
  width: 146px;
  height: 126px;
  -webkit-box-shadow: 0 3px 6px rgba(0, 0, 0, 0.16);
  box-shadow: 0 3px 6px rgba(0, 0, 0, 0.16);
  -webkit-transition: .3s;
  transition: .3s;
  z-index: 10
}

.WebCalender .calender-wrap .days-wrap .-today .day {
  background-color: #7979FF;
  border-radius: 8px;
  color: #fff
}

.modal-wrap {
  position: absolute;
  width: 146px;
  height: 126px;
}

.modal-wrap .content-wrap {
  background-color: #fff;
  border-radius: 5px;
  width: 100%;
  height: 100%;
  padding: 6px 2px;
  -webkit-transition: .3s;
  transition: .3s;
  position: absolute;
  -webkit-box-shadow: 0 3px 6px #00000029;
  box-shadow: 0 3px 6px #00000029;
  padding: 14px 10px 10px;
  z-index: 100
}

.modal-wrap .content-wrap .day {
  display: block;
  color: #666;
  font-size: 12px;
  font-family: "Roboto Condensed", sans-serif;
  text-align: center;
  line-height: 16px;
  margin: 0 auto 3px;
  width: 16px;
  z-index: 10;
  color: #333;
  background-color: #fff;
  font-size: 14px;
  margin: 10px 0 14px;
  width: 100%
}

.modal-wrap .content-wrap .plan {
  position: static;
  height: auto;
  font-size: 14px;
  margin-bottom: 10px
}

.modal-wrap .content-wrap .plan.-long {
  left: -2px;
  background-color: #EA7979;
  border-radius: 5px;
  color: #fff;
  padding: 3px 10px;
  width: 100%;
  z-index: 40;
  display: block;
  position: static;
  padding: 5px 10px
}

.modal-wrap .content-wrap .plan.-long.-red {
  background-color: #EA7979
}

.modal-wrap .content-wrap .plan.-long.-orange {
  background-color: #F8A748
}

.modal-wrap .content-wrap .plan.-long.-yellow {
  background-color: #F2E32D
}

.modal-wrap .content-wrap .plan.-long.-light-green {
  background-color: #88D64E
}

.modal-wrap .content-wrap .plan.-long.-green {
  background-color: #29C48F
}

.modal-wrap .content-wrap .plan.-long.-sky-blue {
  background-color: #6FD2F0
}

.modal-wrap .content-wrap .plan.-long.-blue {
  background-color: #7979FF
}

.modal-wrap .content-wrap .plan.-long.-purple {
  background-color: #BB76E6
}

.modal-wrap .content-wrap .plan.-short {
  display: block;
  width: 100%;
  overflow: hidden;
  padding: 3px 0;
  text-overflow: ellipsis;
  white-space: nowrap;
  white-space: normal;
  text-overflow: clip;
  overflow: visible
}

.modal-wrap .content-wrap .plan.-short:before {
  content: '';
  background-color: #EA7979;
  border-radius: 4px;
  padding: 2px 3px;
  margin-right: 4px;
  height: 8px;
  width: 8px
}

.modal-wrap .content-wrap .plan.-short.-red:before {
  background-color: #EA7979
}

.modal-wrap .content-wrap .plan.-short.-orange:before {
  background-color: #F8A748
}

.modal-wrap .content-wrap .plan.-short.-yellow:before {
  background-color: #F2E32D
}

.modal-wrap .content-wrap .plan.-short.-light-green:before {
  background-color: #88D64E
}

.modal-wrap .content-wrap .plan.-short.-green:before {
  background-color: #29C48F
}

.modal-wrap .content-wrap .plan.-short.-sky-blue:before {
  background-color: #6FD2F0
}

.modal-wrap .content-wrap .plan.-short.-blue:before {
  background-color: #7979FF
}

.modal-wrap .content-wrap .plan.-short.-purple:before {
  background-color: #BB76E6
}

.modal-wrap .content-wrap .plan.-short .time {
  font-family: "Roboto Condensed", sans-serif;
  font-weight: bold;
  color: #999;
  margin-right: 6px
}

.modal-wrap .content-wrap .plan.-short:before {
  padding: 3px 3px
}

.modal-wrap .content-wrap .back-btn {
  color: #aaa;
  position: absolute;
  bottom: 0;
  right: 0;
  cursor: pointer;
  font-family: "Material Design Icons";
  font-size: 20px;
  padding: 10px
}

.modal-wrap.-mult:hover .content-wrap {
  cursor: default;
  height: 300px
}

.modal-wrap:after {
  display: none
}

</style>
