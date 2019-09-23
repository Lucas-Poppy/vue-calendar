<template>
  <div class="container">
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
          <div v-for="day in calendarData" :key="day.index" class="item" :class="{'-weekstart':isWeekStart }">
            <div class="content-wrap">
              <span class="day">{{dayFormat(day)}}</span>
              <span v-for="plan in longPlans(day)" :key="plan.index" class="plan -long -blue" :class="longPlanStatus(plan)">
                <span>{{plan.name}}</span>
              </span>
              <span v-for="plan in dayPlans(day)" :key="plan.index" class="plan -short" :class="colorClass(plan.index)">
                <span class="time">{{plan['start_time']}}</span>{{plan.name}}
              </span>
            </div>
          </div>
          <!-- <div class="item -weekstart -mult -today">
            <div class="content-wrap">
              <span class="day">1</span>
              <span class="plan -long -start -blue">
                <span>夏期講習</span>
              </span>
              <span class="plan -short -red"><span class="time">19:00</span>新規ユーザー...</span>
              <span class="plan -short -purple"><span class="time">20:00</span>小林ゼミナー...</span>
              <span class="plan -short -yellow"><span class="time">21:00</span>生徒さん親さ...</span>
              <span class="mult-num">3</span>
            </div>
          </div>
          <div class="item">
            <div class="content-wrap">
              <span class="day">2</span>
              <span class="plan -long -middle -blue">
                <span>夏期講習</span>
              </span>
            </div>
          </div>
          <div class="item">
            <div class="content-wrap">
              <span class="day">3</span>
              <span class="plan -long -middle -blue">
                <span>夏期講習</span>
              </span>
            </div>
          </div>
          <div class="item">
            <div class="content-wrap">
              <span class="day">4</span>
              <span class="plan -long -middle -blue">
                <span>夏期講習</span>
              </span>
            </div>
          </div>
          <div class="item">
            <div class="content-wrap">
              <span class="day">5</span>
              <span class="plan -long -middle -blue">
                <span>夏期講習</span>
              </span>
            </div>
          </div>
          <div class="item">
            <div class="content-wrap">
              <span class="day">6</span>
              <span class="plan -long -middle -blue">
                <span>夏期講習</span>
              </span>
              <span class="plan -short -red">新規ユーザー訪問</span>
              <span class="plan -short -purple">小林ゼミナール...</span>
            </div>
          </div>
          <div class="item">
            <div class="content-wrap">
              <span class="day">7</span>
              <span class="plan -long -end -blue">
                <span>夏期講習</span>
              </span>
            </div>
          </div> -->
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import moment from 'moment'
export default {
  data () {
    return {
      selectedDate: null, // 今選択している日付,
      current: 1,
      long_plans: [],
      day_plans: [],
      color: ['-red', '-orange', '-yellow', '-light-green', '-green', '-sky-blue', '-blue', '-purple']
    }
  },
  methods: {
    goNextMonth () {
      this.current++
    },
    goPrevMonth () {
      this.current--
    },
    dayPlans (day) {
      return this.day_plans.filter(x => x.day === day)
    },
    colorClass (number) {
      return this.color[number % 8]
    },
    dayFormat (date) {
      return moment(date, 'YYYYMMDD').format('DD')
    },
    longPlans (day) {
      return this.long_plans.filter(x => (x.start_date <= day && x.end_date >= day))
    },
    /**
     * start || middle || end || alone
     */
    longPlanStatus (longPlan) {
      if (longPlan.start_date === longPlan.end_date) {
        return '-alone'
      }
      return {status: 'status'}
    },
    isWeekStart (date) {
      return moment(date, 'YYYYMMDD').day() === 0
    }
  },
  created () {
    this.long_plans = [
      {start_date: '20191012', end_date: '20191024', title: '長期講習1'}
    ]
    this.day_plans = [
      {day: '20191015', start_time: '19:00', end_time: '21:00', name: '数学'}
    ]
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
    dateFormat (dateObject) {
      return ''
    }
  }
}
</script>

<style scoped>
html, body, div, span, object, iframe,
h1, h2, h3, h4, h5, h6, p, blockquote, pre,
abbr, address, cite, code,
del, dfn, em, img, ins, kbd, q, samp,
small, strong, sub, sup, var,
b, i,
dl, dt, dd, ol, ul, li,
fieldset, form, label, legend,
table, caption, tbody, tfoot, thead, tr, th, td,
article, aside, canvas, details, figcaption, figure,
footer, header, hgroup, menu, nav, section, summary,
time, mark, audio, video {
  margin: 0;
  padding: 0;
  border: 0;
  outline: 0;
  font-size: 100%;
  vertical-align: baseline;
  background: transparent; }

body {
  line-height: 1; }

article, aside, details, figcaption, figure,
footer, header, hgroup, menu, nav, section {
  display: block; }

nav ul {
  list-style: none; }

blockquote, q {
  quotes: none; }

blockquote:before, blockquote:after,
q:before, q:after {
  content: '';
  content: none; }

a {
  margin: 0;
  padding: 0;
  font-size: 100%;
  vertical-align: baseline;
  background: transparent; }

/* change colours to suit your needs */
ins {
  background-color: #ff9;
  color: #000;
  text-decoration: none; }

/* change colours to suit your needs */
mark {
  background-color: #ff9;
  color: #000;
  font-style: italic;
  font-weight: bold; }

del {
  text-decoration: line-through; }

abbr[title], dfn[title] {
  border-bottom: 1px dotted;
  cursor: help; }

table {
  border-collapse: collapse;
  border-spacing: 0; }

/* change border colour to suit your needs */
hr {
  display: block;
  height: 1px;
  border: 0;
  border-top: 1px solid #cccccc;
  margin: 1em 0;
  padding: 0; }

input, select {
  vertical-align: middle; }

li {
  list-style: none; }
*,
*:after,
*:before {
  -webkit-box-sizing: border-box;
  box-sizing: border-box;
  color: #333;
  font-family: 'Hiragino Kaku Gothic ProN', 'ヒラギノ角ゴ ProN W3', sans-serif
}

.container {
  width: 1080px;
  height: 100vh;
  margin: 0 auto;
  border-left: solid 1px #ccc;
  border-right: solid 1px #ccc;
  padding: 64px calc((1080px - 986px) / 2) 0
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
  margin: 2px;
  width: 136px;
  height: 116px
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

.WebCalender .calender-wrap .days-wrap .item .plan {
  font-size: 12px;
  margin-bottom: 2px
}

.WebCalender .calender-wrap .days-wrap .item .plan.-short {
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

.WebCalender .calender-wrap .days-wrap .item .plan.-long {
  position: relative;
  display: block;
  width: 100%;
  height: calc(1em + 6px)
}

.WebCalender .calender-wrap .days-wrap .item .plan.-long span {
  background-color: #EA7979;
  color: #fff;
  padding: 3px 10px;
  border-radius: 5px;
  position: absolute;
  left: -2px;
  z-index: 10
}

.WebCalender .calender-wrap .days-wrap .item .plan.-long.-red span {
  background-color: #EA7979;
  color: #EA7979
}

.WebCalender .calender-wrap .days-wrap .item .plan.-long.-orange span {
  background-color: #F8A748;
  color: #F8A748
}

.WebCalender .calender-wrap .days-wrap .item .plan.-long.-yellow span {
  background-color: #F2E32D;
  color: #F2E32D
}

.WebCalender .calender-wrap .days-wrap .item .plan.-long.-light-green span {
  background-color: #88D64E;
  color: #88D64E
}

.WebCalender .calender-wrap .days-wrap .item .plan.-long.-green span {
  background-color: #29C48F;
  color: #29C48F
}

.WebCalender .calender-wrap .days-wrap .item .plan.-long.-sky-blue span {
  background-color: #6FD2F0;
  color: #6FD2F0
}

.WebCalender .calender-wrap .days-wrap .item .plan.-long.-blue span {
  background-color: #7979FF;
  color: #7979FF
}

.WebCalender .calender-wrap .days-wrap .item .plan.-long.-purple span {
  background-color: #BB76E6;
  color: #BB76E6
}

.WebCalender .calender-wrap .days-wrap .item .plan.-long.-start span {
  border-radius: 5px 0 0 5px;
  color: #fff;
  width: calc(140px - 2px)
}

.WebCalender .calender-wrap .days-wrap .item .plan.-long.-middle span {
  width: 140px;
  left: -4px;
  border-radius: unset
}

.WebCalender .calender-wrap .days-wrap .item .plan.-long.-end span {
  border-radius: 0 5px 5px 0;
  left: -4px;
  width: calc(140px - 2px)
}

.WebCalender .calender-wrap .days-wrap .item .plan.-long.-alone span {
  width: calc(140px - 4px)
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
  z-index: 30
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
  transition: .3s
}

.WebCalender .calender-wrap .days-wrap .item.-open .content-wrap {
  position: absolute;
  min-width: 245px;
  height: 300px;
  -webkit-box-shadow: 0 3px 6px #00000029;
  box-shadow: 0 3px 6px #00000029;
  padding: 14px 10px 10px;
  z-index: 30
}

.WebCalender .calender-wrap .days-wrap .item.-open .content-wrap .day {
  color: #333;
  background-color: #fff;
  font-size: 14px;
  margin: 10px 0 14px;
  width: 100%
}

.WebCalender .calender-wrap .days-wrap .item.-open .content-wrap .plan {
  font-size: 14px;
  margin-bottom: 10px
}

.WebCalender .calender-wrap .days-wrap .item.-open .content-wrap .plan.-long {
  display: none
}

.WebCalender .calender-wrap .days-wrap .item.-open .content-wrap .plan.-long-op {
  color: #fff;
  display: block;
  width: 100%;
  border-radius: 5px;
  padding: 6px 14px
}

.WebCalender .calender-wrap .days-wrap .item.-open .content-wrap .plan.-long-op.-red {
  background-color: #EA7979
}

.WebCalender .calender-wrap .days-wrap .item.-open .content-wrap .plan.-long-op.-orange {
  background-color: #F8A748
}

.WebCalender .calender-wrap .days-wrap .item.-open .content-wrap .plan.-long-op.-yellow {
  background-color: #F2E32D
}

.WebCalender .calender-wrap .days-wrap .item.-open .content-wrap .plan.-long-op.-light-green {
  background-color: #88D64E
}

.WebCalender .calender-wrap .days-wrap .item.-open .content-wrap .plan.-long-op.-green {
  background-color: #29C48F
}

.WebCalender .calender-wrap .days-wrap .item.-open .content-wrap .plan.-long-op.-sky-blue {
  background-color: #6FD2F0
}

.WebCalender .calender-wrap .days-wrap .item.-open .content-wrap .plan.-long-op.-blue {
  background-color: #7979FF
}

.WebCalender .calender-wrap .days-wrap .item.-open .content-wrap .plan.-long-op.-purple {
  background-color: #BB76E6
}

.WebCalender .calender-wrap .days-wrap .item.-open .content-wrap .plan.-short:before {
  padding: 3px 3px
}

.WebCalender .calender-wrap .days-wrap .item.-open .content-wrap .back-btn {
  color: #aaa;
  position: absolute;
  bottom: 0;
  right: 0;
  cursor: pointer;
  font-family: "Material Design Icons";
  font-size: 20px;
  padding: 10px
}

.WebCalender .calender-wrap .days-wrap .item.-open.-mult:hover .content-wrap {
  cursor: default;
  height: 300px
}

.WebCalender .calender-wrap .days-wrap .item.-open:after {
  display: none
}

.WebCalender .calender-wrap .days-wrap .item.-weekstart .plan.-long span {
  color: #fff
}

.WebCalender .calender-wrap .days-wrap .-today .day {
  background-color: #7979FF;
  border-radius: 8px;
  color: #fff
}

</style>
