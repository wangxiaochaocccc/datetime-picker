<template>
    <div :style='{width:width}' class='date-range-container' @click.stop="chooseDateRange" v-on:click='calendarClicked($event)'>
      <div class="date-box" id="tj-datetime-input1">
        <input ref="inputDomL" :class="{borderB:isborder1}" type="text" :value="this.date[0]">
        <span>~</span>
        <input ref="inputDomR" class="inputR" :class="{borderB:isborder2}" type="text" :value="this.date[1]">
      </div>
      <div>
        <div class='calender-div' :class='{noDisplay: hideCal}'>
          <div class="main-time-box">
            <div :class='{noDisplay: hideDate}' class="left-box">
              <div class='year-month-wrapper'>
                <div class='month-setter'>
                  <button type='button' class='nav-l' v-on:click.stop='leftYear' >&#x3C;</button>
                  <span class='year'>{{year}}</span>
                  <button type='button' class='nav-r' v-on:click.stop='rightYear' >&#x3E;</button>
                </div>
                <div class='month-setter'>
                  <button type='button' class='nav-l' v-on:click.stop='leftMonth' >&#x3C;</button>
                  <span class='month'>{{month}}</span>
                  <button type='button' class='nav-r' v-on:click.stop='rightMonth'>&#x3E;</button>
                </div>
              </div>
              <div class='headers'>
                <span class='days' v-for="port in days" :key="port">{{port}}</span>
              </div>
              <div class="day-box">
                <div class="week" v-for="(week, weekIndex) in weeks" :key="weekIndex">
                  <span :data1="dayIndexAll" class="port" v-for="(day, dayIndex) in week" :key="dayIndex" v-on:click.stop='setDay(weekIndex*7 + dayIndex, day)' :class='{activePort: (weekIndex*7 + dayIndex) === activePort,noPointer:(isNoPorinter&&(weekIndex*7 + dayIndex) < dayIndexAll&&monthIndex <=isNoPorinterMonthindex&&year<=isNoPorinterYear)||banDate}'>
                    {{day}}
                  </span>
                </div>
              </div>
            </div>
            <div class="right-box">
              <div class='time-picker' :class='{noDisplay: hideTime}'>
                <div class='hour-selector' >
                  <div v-on:click.stop='showHourSelector' id='j-hour'>{{periodStyle === 12 && hour > 12 ? hour - 12 : hour}}</div>
                </div>
                <div class='time-separator'>
                  <span>:</span>
                </div>
                <div class='minute-selector' >
                  <div v-on:click.stop='showMinuteSelector' id='j-minute'>{{minute}}</div>
                </div>
                <div class='time-separator'>
                  <span>:</span>
                </div>
                <!-- <div class='minute-selector' v-if='periodStyle === 12'>
                  <div v-on:click.stop='changePeriod'>{{period}}</div>
                </div> -->
                <!-- <div class='time-separator' v-if='periodStyle === 12'>
                  <span>:</span>
                </div> -->
                <div class='minute-selector'>
                  <div>{{second}}</div>
                </div>
              </div>
              <div class="choose-box">
                  <div class='scroll-hider' ref='hourScrollerWrapper' :class='{showSelector: hourSelectorVisible}'>
                    <ul ref='hourScroller'>
                      <li v-for="(h, index) in hours" :key="index" :class='{active: index === hourIndex}' v-on:click.stop='setHour(index, true)' >{{h}}</li>
                    </ul>
                  </div>
                  <div class='scroll-hider' ref='minuteScrollerWrapper' :class='{showSelector: minuteSelectorVisible}'>
                    <ul ref='minuteScroller'>
                      <li v-for="(m, index) in minutes" :key="index" :class='{active: index === minuteIndex}' v-on:click.stop='setMinute(index, true)'>{{m}}</li>
                    </ul>
                  </div>
                  <div class='scroll-hider' ref='secondScrollerWrapper' :class='{showSelector: minuteSelectorVisible}'>
                    <ul ref='secondScroller'>
                      <li v-for="(m, index) in minutes" :key="index" :class='{active: index === secondsIndex}' v-on:click.stop='setSeconds(index, true)'>{{m}}</li>
                    </ul>
                  </div>
              </div>
            </div>
          
          </div>
          <button type='button' v-on:click.stop='clearDate' class='okButton'>清除</button>
          <button type='button' v-on:click.stop='setDate' class='okButton ok'>确定</button>
        </div>
      </div>
    </div>
</template>

<script>
import endOfMonth from 'date-fns/end_of_month';
import eachDay from 'date-fns/each_day';
import getDay from 'date-fns/get_day';
import format from 'date-fns/format';
import startOfDay from 'date-fns/start_of_day';
import isEqual from 'date-fns/is_equal';

const days = ['日', '一', '二', '三', '四', '五', '六'];
const AM = 'AM'
const PM = 'PM'
export default {
  name: 'datetime-picker',
  props: {
    format: {
      type: String,
      default: 'YYYY-MM-DD h:i:s',
    },
    name: {
      type: String
    },
    width: {
      type: String
    },
    value: {
      type: Array,
      default: []
    },
    required: {
      type: Boolean,
      default: false
    },
    readonly: {
      type: Boolean,
      default: false
    },
    firstDayOfWeek: {
      default: 0,
      validator: function(value) {
        try {
          const val = parseInt(value, 10)
          return val >= 0 && val <= 1
        } catch (e) {
          console.warn(e.message)
          return false
        }
      },
      message: 'Only 0 (Sunday) and 1 (Monday) are supported.'
    },
  },
  data () {
    return {
      date:this.value,
      isStartTime:true,
      hideCal: true,
      activePort: null,
      timeStamp: new Date(),
      months: ['一月', '二月', '三月', '四月', '五月', '六月', '七月', '八月', '九月', '十月', '十一月', '十二月'],
      days: [],
      weeks:[],
      monthIndex: 0,
      dayIndexAll:0,
      hourIndex: 0,
      minuteIndex: 0,
      year: 2023,
      portsHolder: [],
      minute: '00',
      hour: '00',
      day: 1,
      minuteSelectorVisible: false,
      hourSelectorVisible: false,
      period: AM,
      periodStyle: 12,
      monthStr: '',
      isborder1:false,
      isborder2:false,
      isNoPorinter:false,
      second:'00',
      secondsIndex:0,
      isNoPorinterMonthindex:0,
      isNoPorinterYear:null
    }
  },
  // mounted() {
  //   this.months = ['Янв', 'Фев', 'Мар', 'Апр', 'Maй', 'Июн', 'Июл', 'Авг', 'Сен', 'Окт', 'Ноя', 'Дек'];
  // },
  methods: {
    initDate(val){
      if (this.value.length) {
        try {
          this.timeStamp = this.makeDateObject(val)

          // set #period (am or pm) based on date hour
          if (this.timeStamp.getHours() >= 12) {
            this.period = PM
          } else {
            this.period = AM
          }
        } catch (e) {
          this.timeStamp = new Date()
          console.log(e);
        }
      }
      this.year = this.timeStamp.getFullYear()
      this.monthIndex = this.timeStamp.getMonth()
      this.day = this.timeStamp.getDate()
      if(!this.isNoPorinter){
        this.dayIndexAll = this.day
      }
      if(this.value.length) {
        this.hour = this.timeStamp.getHours()
      this.hourIndex = this.hour
      this.hour = this.hour < 10 ? '0' + this.hour : '' + this.hour
      this.minute = this.timeStamp.getMinutes()
      this.minuteIndex = this.minute
      this.minute = this.minute < 10 ? '0' + this.minute : '' + this.minute
      this.second = this.timeStamp.getSeconds()
      this.secondsIndex = this.second
      this.second = this.second < 10 ? '0' + this.second : '' + this.second
      }
      this.updateCalendar()
      if(this.value.length){
        this.setHour(this.hourIndex,true)
        this.setMinute(this.minuteIndex,true)
        this.setSeconds(this.secondsIndex)
      }
      if(this.$refs.hourScroller){
        this.scrollTopHour()
      }
      if(this.$refs.minuteScroller){
        this.scrollTopSecond()
      }
      if(this.$refs.secondScroller){
        this.scrollTopMinute()
      }
      days.forEach((day, idx) => {
        this.days[(idx - this.normalizedFirstDayOfWeek + 7) % 7] = day;
      });
      document.addEventListener('keydown', this.keyIsDown)
      document.addEventListener('click', this.documentClicked)
      // this.setDate()
      this.setPeriodStyle()
    },
    leftMonth () {
      let index = this.months.indexOf(this.month)
      if (index === 0) {
        this.monthIndex = 11
      } else {
        this.monthIndex = index - 1
      }
      this.updateCalendar()
    },
    rightMonth () {
      let index = this.months.indexOf(this.month)
      if (index === 11) {
        this.monthIndex = 0
      } else {
        this.monthIndex = index + 1
      }
      this.updateCalendar()
    },
    rightYear () {
      this.year++
      this.updateCalendar()
    },
    leftYear () {
      this.year--
      this.updateCalendar()
    },
    updateActivePortFromWeek (week, weekIndex) {
      const currentActive = startOfDay(this.timeStamp);
      const index = week.findIndex(day => isEqual(currentActive, day));
      if (index !== -1) {
        this.activePort = weekIndex*7 + index;
      }
    },
    updateCalendar () {
      const date = new Date(this.year, this.monthIndex, 1, 0, 0, 0);
      const weeks = [];
      let week = null;
      let weekDays = new Array(7);
      // let index = 0;
      this.activePort = null;
      eachDay(date, endOfMonth(date)).forEach(day => {
        const weekday = getDay(day);
        if (weekday === this.normalizedFirstDayOfWeek) {
          if (week) {
            weeks.push(week);
            // Add those days that were not part of the month to the index
            // index += week.filter(d => d === null).length;
            this.updateActivePortFromWeek(weekDays, weeks.length - 1);
            weekDays = new Array(7);
          }
          week = new Array(7);
        } else if (week === null) {
          week = new Array(7);
        }
        const idx = (weekday - this.normalizedFirstDayOfWeek + 7) % 7
        week[idx] = format(day, 'DD');
        weekDays[idx] = day;
      });
      if (week && week.some(n => n)) {
        weeks.push(week);
        this.updateActivePortFromWeek(weekDays, weeks.length - 1);
      }
      this.weeks = weeks;
    },
    setDay (index, port) {
      if(!this.isNoPorinter){
        this.dayIndexAll = index
      }
      if (port) {
        this.activePort = index;
        this.day = parseInt(port, 10);
        this.timeStamp = new Date(this.year, this.monthIndex, this.day);
      }
    },
    setMinute (index, closeAfterSet) {
      this.minuteIndex = index
      this.minute = this.minutes[index]
      if (closeAfterSet) {
        this.minuteSelectorVisible = false
      }
    },
    setSeconds(index,closeAfterSet){
      this.secondsIndex = index
      this.second = this.minutes[index]
    },
    setHour (index, closeAfterSet) {
      this.hourIndex = index
      this.hour = this.hours[index]
      if (closeAfterSet) {
        this.hourSelectorVisible = false
      }
    },
    showHourSelector () {
      this.hourSelectorVisible = true
      this.minuteSelectorVisible = false
    },
    showMinuteSelector () {
      this.minuteSelectorVisible = true
      this.hourSelectorVisible = false
    },
    keyIsDown (event) {
      let key = event.which || event.keycode
      if (key === 38) {
        if (this.minuteSelectorVisible && this.minuteIndex > 0) {
          this.setMinute(this.minuteIndex - 1, false)
          this.scrollTopMinute()
        } else if (this.hourSelectorVisible && this.hourIndex > 0) {
          this.setHour(this.hourIndex - 1, false)
          this.scrollTopHour()
        }
      } else if (key === 40) {
        if (this.minuteSelectorVisible && this.minuteIndex < this.minutes.length - 1) {
          this.setMinute(this.minuteIndex + 1, false)
          this.scrollTopMinute()
        } else if (this.hourSelectorVisible && this.hourIndex < this.hours.length - 1) {
          this.setHour(this.hourIndex + 1, false)
          this.scrollTopHour()
        }
      } else if (key === 13) {
        this.minuteSelectorVisible = false
        this.hourSelectorVisible = false
      }
    if (this.minuteSelectorVisible || this.hourSelectorVisible) {
    event.preventDefault()
    this.minuteSelectorVisible = false
        this.hourSelectorVisible = false
    }
    },
    scrollTopMinute () {
      let mHeight = this.$refs.minuteScroller.scrollHeight || this.minuteIndex*30
      let wHeight = this.$refs.minuteScrollerWrapper.clientHeight
      let top = mHeight * (this.minuteIndex / (this.minutes.length - 1)) - (wHeight / 2)
      this.$refs.minuteScroller.scrollTop = top
    },
    scrollTopSecond () {
      let mHeight = this.$refs.secondScroller.scrollHeight || this.secondsIndex*30
      let wHeight = this.$refs.secondScrollerWrapper.clientHeight
      let top = mHeight * (this.secondsIndex / (this.minutes.length - 1)) - (wHeight / 2)
      this.$refs.secondScroller.scrollTop = top
    },
    scrollTopHour () {
      let mHeight =this.$refs.hourScroller.scrollHeight
      let wHeight = this.$refs.hourScrollerWrapper.clientHeight
      let top = mHeight * (this.hourIndex / (this.hours.length - 1)) - (wHeight / 2)
      console.log(top);
      this.$refs.hourScroller.scrollTop = top
    },
    changePeriod () {
      this.period = this.period === AM ? PM : AM
    },
    calendarClicked (event) {
      if (event.target.id !== 'j-hour' && event.target.id !== 'j-minute') {
        this.minuteSelectorVisible = false
        this.hourSelectorVisible = false
      }
      event.cancelBubble = true
      if (event.stopPropagation) {
        event.stopPropagation()
      }
    },
    documentClicked (event) {
      if (event.target.id !== 'tj-datetime-input1') {
        this.hideCal = true
        this.isborder1 = false
        this.isborder2 = false
        this.isNoPorinter = false
        this.isStartTime = true
      }
    },
    // toggleCal () {
    //   this.hideCal = !this.hideCal
    // },
    setPeriodStyle () {
      if (this.dateFormat.indexOf('H') !== -1) {
        this.periodStyle = 24;
        this.period = null;
      } else {
        this.periodStyle = 12;
      }
    },
    setDate () {
      let d = null

      this.setPeriodStyle()

      let h = this.hour + ''

      if (this.periodStyle === 12) {
        if (h === '12') {
          h = this.period === AM ? '00' : '12'
        } else if (this.period === PM && parseInt(h) < 12) {
          h = parseInt(h) + 12
          h = '' + h
        } else if (this.period === AM && parseInt(h) > 12) {
          h = parseInt(h) - 12
          h = '' + h
        }
      }
      d = this.dateFormat
      console.log(this.year,this.day,this.monthIndex,this.hour);
      d = d.replace('YYYY', this.year)
      d = d.replace('DD', this.day < 10 ? '0' + this.day : this.day)
      let m = this.monthIndex + 1
      d = d.replace('MM', m < 10 ? '0' + m : m)
      this.minute += ''
      d = d.replace(this.periodStyle === 24 ? 'H' : 'h', h.length < 2 ? '0' + h : '' + h )
      d = d.replace('i', this.minute.length < 2 ? '0' + this.minute : '' + this.minute)
      d = d.replace('s', this.second < 10 ? this.second : this.second)
      // this.$emit('input', d)
      this.saveChooseTime(d)
      console.log(this.date);
    },
    saveChooseTime (time) {
      if (this.isStartTime) {
        this.isStartTime = false
        this.date[0] = time
        this.$refs.inputDomR.focus()
        this.isborder2 = true
        this.isborder1 = false
        this.isNoPorinter = true
        this.isNoPorinterMonthindex = this.monthIndex
        this.isNoPorinterYear = this.year
        if(this.compareDate(this.date[0],this.date[1])){
          this.date[1] = null
        }
        if(this.date[1]){
          this.initDate(this.date[1])
        }
      } else {
        this.date[1] = time
        this.hideCal = true
        this.isNoPorinter = false
        this.isStartTime = true
        this.isborder2 = false
      }
    },
    /**
    `*Creates a date object from a given date string
    */
    makeDateObject (val) {
      // handle support for eu date format
      let dateAndTime = val.split(' ')
      let arr = []
      if (this.format.indexOf('-') !== -1) {
        arr = dateAndTime[0].split('-')
      } else {
        arr = dateAndTime[0].split('/')
      }
      let year = 0
      let month = 0
      let day = 0
      if (this.format.indexOf('DD/MM/YYYY') === 0 || this.format.indexOf('DD-MM-YYYY') === 0) {
        year = arr[2]
        month = arr[1]
        day = arr[0]
      } else if (this.format.indexOf('YYYY/MM/DD') === 0 || this.format.indexOf('YYYY-MM-DD') === 0) {
        year = arr[0]
        month = arr[1]
        day = arr[2]
      } else {
        year = arr[2]
        month = arr[0]
        day = arr[1]
      }

      let date = new Date();
      if(this.hideDate){
        // time only
        let splitTime = dateAndTime[0].split(':')
        // handle date format without seconds
        let secs = splitTime.length > 2 ? parseInt(splitTime[2]) : 0
        date.setHours(parseInt(splitTime[0]), parseInt(splitTime[1]), secs, 0)
      } else if (this.hideTime) {
        // date only
        date = new Date(parseInt(year), parseInt(month)-1, parseInt(day))
      } else {
        // we have both date and time
        let splitTime = dateAndTime[1].split(':')
        // handle date format without seconds
        let secs = splitTime.length > 2 ? parseInt(splitTime[2]) : 0
        date = new Date(parseInt(year), parseInt(month)-1, parseInt(day), parseInt(splitTime[0]), parseInt(splitTime[1]), secs)
      }

      return date
    },
    clearDate(){
      this.date = []
      // this.$emit('input', '')
      // this.toggleCal ()
    },
    chooseDateRange() {
      if(this.date[0]){
        this.initDate(this.date[0])
      }
      this.hideCal = false
      this.isborder1 = true
      this.$refs.inputDomL.focus()
    },
    // 比较时间大小
    compareDate(date1,date2){
      var oDate1 = new Date(date1);
      var oDate2 = new Date(date2);
      if(oDate1.getTime() > oDate2.getTime()){
        return true; //第一个大
      } else {
        return false; //第二个大
      }
    }
  },
  // created () {
  //   // this.initDate(this.date[0])
  // },
  mounted(){
    this.$nextTick(function(){
      this.initDate(this.date[0])
    })
  },
  watch: {
    // value (newVal, oldVal) {
    //   if (newVal) {
    //     this.value = newVal;
    //     try {
    //       this.timeStamp = this.makeDateObject(this.value)
    //     } catch (e) {
    //       console.warn(e.message +'. Current date is being used.')
    //       this.timeStamp = new Date()
    //     }
    //     this.year = this.timeStamp.getFullYear()
    //     this.monthIndex = this.timeStamp.getMonth()
    //     this.day = this.timeStamp.getDate()
    //     this.hour = this.timeStamp.getHours()
    //     this.hour = this.hour < 10 ? '0' + this.hour : '' + this.hour
    //     this.minute = this.timeStamp.getMinutes()
    //     this.minute = this.minute < 10 ? '0' + this.minute : '' + this.minute
    //     this.updateCalendar()
    //     this.setDate()
    //   }
    // }
  },
  destroyed: function () {
    document.removeEventListener('keydown', this.keyIsDown)
    document.removeEventListener('click', this.documentClicked)
  },
  computed: {
    normalizedFirstDayOfWeek: function() {
      return parseInt(this.firstDayOfWeek, 10);
    },
    ports: {
      get: function () {
        let p = []
        if (this.portsHolder.length === 0) {
          for (let i = 0; i < 42; i++) {
            p.push('')
          }
        } else {
          p = this.portsHolder
        }
        return p
      },
      set: function (newValue) {
        this.portsHolder = newValue
      }
    },
    month () {
      return this.months[this.monthIndex]
    },
    dateTime () {
      return this.timeStamp.getFullYear() + '-' + (this.timeStamp.getMonth() + 1) + '-' + this.timeStamp.getUTCDay()
    },
    minutes () {
      let arr = []
      for (let i = 0; i < 60; i++) {
        i < 10 ? arr.push('0' + i) : arr.push('' + i)
      }
      return arr
    },
    hours () {
      let arr = []
      if (this.periodStyle === 24) {
        for (let i = 0; i < this.periodStyle; i++) {
          i < 10 ? arr.push('0' + i) : arr.push('' + i)
        }
      } else {
        for (let i = 1; i <= this.periodStyle; i++) {
          i < 10 ? arr.push('0' + i) : arr.push('' + i)
        }
      }
      return arr
    },
    dateFormat () {
      let f = 'YYYY-MM-DD h:i:s'
      let allowedFormats = [
        'YYYY-MM-DD h:i:s',
        'YYYY-MM-DD',
        'YYYY/MM/DD',
        'YYYY/MM/DD h:i:s',
        'YYYY/MM/DD H:i:s', 
      ]
      if (this.format) {
        f = this.format
      }
      if (allowedFormats.indexOf(f) < 0) {
        console.warn('Invalid date format supplied. Current default date format is being used.')
        // return default date format if date format is invalid
        return 'YYYY-MM-DD h:i:s'
      } else {
        return f
      }
    },
    hideTime () {
      return this.dateFormat.indexOf('h:i:s') === -1
          && this.dateFormat.indexOf('H:i:s') === -1
          && this.dateFormat.indexOf('h:i') === -1
          && this.dateFormat.indexOf('H:i') === -1
    },
    hideDate () {
      return this.dateFormat === 'h:i:s' || this.dateFormat === 'H:i:s'
        || this.dateFormat === 'h:i' || this.dateFormat === 'H:i'
    },
    banDate(){
      return this.isNoPorinter&&(this.year < this.isNoPorinterYear || (this.monthIndex < this.isNoPorinterMonthindex&&this.year<=this.isNoPorinterYear))
    }
  }
}
</script>
<style lang='scss'>
.date-range-container {
  position: relative;
  height: 32px;
  padding: 0 20px;
  border: 1px solid #ddd;
  background-color: #fff;
  .borderB {
    border-bottom:2px solid #4096ff !important;
  }
  .noPointer {
    pointer-events: none;
    background:#eee;
    color:rgba(0,0,0,0.25)
  }
  .date-box {
    display: flex;
    align-items: center;
    justify-content: space-between;
    input {
      display: inline-block;
      border: 0;
      height: 30px;
      outline: none
    }
  }
  .inputR {
    text-align:right;
  }
  .year-month-wrapper{
    display: flex;
    background-color: #fff;
    border-bottom: 1px solid #ddd;
  }
  .day-box {
    display: flex;
    justify-content:space-between;
    flex-direction:column;
    height: 200px;
  }
  .calender-div{
    min-width: 270px;
    box-shadow: 1px 2px 5px #ccc;
    background: #FFF;
    position: absolute;
    display: inline-block;
    left: 0;
    top: 35px;
    color: #444;
    font-size: 14px;
    padding-bottom: 10px;
    z-index: 100;
  }
  .main-time-box {
    display: flex;
  }
  .left-box {
    width: 240px;
    height: 273px;
    border-right: 1px solid #ddd;
    border-bottom: 1px solid #ddd;
  }
  .choose-box {
    display: flex;
    border-bottom: 1px solid #ddd;
    .scroll-hider {
      /* display: none;
      // vertical-align:top; */
      overflow:hidden;
      // border:0;
      // position: absolute;
      // top: 28px;
    }
    .scroll-hider ul {
      padding:7.2px;
      margin:-5px -13px -5px -5px;
      list-style-type: none;
      height: 234px;
      overflow: auto;
      width:55px;
      color: #999;
      overflow-x: hidden;
    }
    li.active{
      background-color: #fff;
      color: #000;
    }
    li{
      padding: 5px 13px;
      font-size: 14px;
      width: 100%;
      cursor: pointer;
    }
  }
  .port, .days{
    display: inline-block;
    width: 30px;
    height: 30px;
    padding: 5px 3px;
    margin: 2px;
    border-radius: 2px;
    text-align: center;
    vertical-align: top;
    cursor: pointer;
    box-sizing: border-box;
  }
  .days{
    color: #000;
    font-weight: bold;
  }
  .port:hover{
    color: #000;
    font-weight: bold;
  }
  .activePort, .activePort:hover {
    background-color: #1890ff;
    color: white;
  }
  .month-setter, .year-setter{
    margin: 0 1px;
    width: 48.2%;
    color: white;
    font-weight: 900;
    display: flex;
    justify-content: space-between;
  }
  .nav-l:hover, .nav-r:hover {
    background-color: #fff;
  }
  .nav-l, .nav-r {
    display: inline-block;
    width: 25px;
    background-color: #fff;
    color: #000;
    font-size: 16px;
    cursor: pointer;
    border: 0;
    padding: 7px;
    margin:0;
  }
  .nav-l:focus, .nav-r:focus{
    outline: none;
  }
  .nav-l{
    float: left;
  }
  .nav-r{
    float: right;
  }
  .month, .year{
    width: 60px;
    text-align: center;
    display: inline-block;
    color: #000;
    padding: 7px 0;
  }
  /* .headers>span{

  } */
  .hour-selector, .minute-selector{
    width: 30px;
    display: inline-block;
    text-align: center;
    font-weight: bold;
    position: relative;
    cursor: pointer;
  }
  .time-separator{
    display: inline-block;
    font-weight: bold;
  }
  .time-picker{
    /* margin: 10px */
    // width: 100px;
  }
  .nav-t, .nav-d{
    font-weight: bold;
    cursor: pointer;
  }
  .showSelector{
    display:inline-block;
  }
  
  .time-picker{
    display: flex;
    align-items: center;
    justify-content: center;
    height: 34px;
    border-bottom: 1px solid #ddd;
  }
  .noDisplay{
    visibility: hidden;
  }
  .okButton{
    color: #000;
    font-size: 15px;
    font-weight: bold;
    padding: 0;
    float: right;
    border: 0;
    margin-right: 10px;
    margin-top: 10px;
    cursor: pointer;
    background: transparent;
  }
}
</style>