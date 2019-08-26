<template>
  <div id="app">
    <div>日期:{{curdate}}</div>
    <button @click="toPrevMonth(curdate)">上个月</button>
    <button @click="toNextMonth(curdate)">下个月</button>
    <calendar @calendarData="geCalendar" ref="calendar"/>
  </div>
</template>

<script>
import calendar from './components/calendar.vue'

export default {
  name: 'app',
  components: {
    calendar
  },
  data(){
    return{
      calendar:'',
      eventlist:[
        { msg: "校庆日", date: "2019-8-20" },
        { msg: "校庆日2", date: "2019-8-2" },
        { msg: "校庆日3", date: "2019-9-21" },
        { msg: "校庆日", date: "2019-8-20" },
        { msg: "校庆日", date: "2019-8-20" },
        { msg: "校庆日", date: "2019-8-20" },
        { msg: "校庆日", date: "2019-8-20" },
      ]
    }
  },
  computed:{
    curdate(){
      return this.calendar.curdate
    }
  },
  methods:{
    geCalendar(val){
      this.calendar=val
    },
    toPrevMonth(curdate){
    let that=this
  if(curdate[1]===1){
    that.curdate[0]--
    that.curdate[1]=12
  }else{
     that.curdate[1]--
  }
let prev=that.curdate.join('/')
that.$refs.calendar.init(prev)
window.console.log(this.calendar)
    },
    toNextMonth(curdate){
    let that=this
  if(curdate[1]===12){
    that.curdate[0]++
    that.curdate[1]=1
  }else{
     that.curdate[1]++
  }
let next=that.curdate.join('/')
that.$refs.calendar.init(next)
window.console.log(that.calendar)
    }
  },
  mounted(){
    window.console.log(this.calendar)
  }
}
</script>

<style>
body{
  margin: 0;
}
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
