<template>
  <div class="calendar">
    <div class="calendar-title">
      <span :class="['title-item',{'weekend':index===5||index===6}]" v-for="(item,index) in calendar.titles" :key="index">{{item}}</span>
    </div>
    <div class="calendar-content">
      <div
        v-for="(item,index) in calendar.cArr"
        :class="['day-block',{'active':calendar.contents[index].active}]"
        :key="index"
      >
        {{item | addZero}}
        <div
          v-for="(item,index) in calendar.contents[index].msg"
          :key="index"
          :v-if="calendar.contents[index].msg"
          class="event"
        >
          {{item.text}}
          <br />
          {{item.tips?item.tips:''}}
          <br />
          {{item.time?item.time:''}}
        </div>
        <div
          v-for="(item,index) in calendar.contents[index].festival"
          :key="index"
          :v-if="calendar.contents[index].festival"
          class="festival"
        ><strong>{{item.name}}</strong><br>{{item.vacation?'（法定节假日）':''}}</div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "calendar",
  props: {
    msg: String
  },
  data() {
    return {
      calendar: {
        curdate: "",
        titles: [
          "星期一",
          "星期二",
          "星期三",
          "星期四",
          "星期五",
          "星期六",
          "星期天"
        ],
        monthfirstdayweek: "",
        prevmonthlen: "",
        monthlen: "",
        nextmonthlen: "",
        cArr: "",
        contents: [],
        evenList: [
          {
            msg: [
              { text: "春季音乐会", tips: "合唱及弦乐", time: "7:00-9:00" }
            ],
            date: "2019-8-20"
          }
        ],
        festival: [
          { date: "1-1", festival: [{name:"元旦节",vacation:true}] },
          { date: "2-14", festival: [{name:"情人节"}] },
          { date: "3-8", festival: [{name:"妇女节"}] },
          { date: "4-1", festival: [{name:"愚人节"}] },
          { date: "5-1", festival: [{name:"劳动节",vacation:true}] },
          { date: "5-2", festival: [{name:"劳动节",vacation:true}] },
          { date: "5-3", festival: [{name:"劳动节",vacation:true}] },
          { date: "5-4", festival: [{name:"青年节"}] },
          { date: "6-1", festival: [{name:"六一儿童节"}] },
          { date: "7-1", festival: [{name:"建党节"}] },
          { date: "8-1", festival: [{name:"建军节"}] },
          { date: "9-10", festival: [{name:"教师节",}] },
          { date: "10-1", festival: [{name:"国庆节",vacation:true}]},
          { date: "10-2", festival: [{name:"国庆节",vacation:true}]},
          { date: "10-3", festival: [{name:"国庆节",vacation:true}]},
          { date: "11-1", festival: [{name:"万圣夜"}] },
          { date: "11-24", festival: [{name:"平安夜"}] },
          { date: "11-25", festival: [{name:"圣诞节"}] }
        ]
      }
    };
  },
  filters: {
    addZero(val) {
      if (val < 10) {
        val = "0" + val;
      }
      return val;
    }
  },
  methods: {
    init(val) {
      let that = this;
      let calendar = that.calendar;
      let date = "";
      if (val) {
        date = new Date(val).toLocaleDateString();
      } else {
        date = new Date().toLocaleDateString();
      }
      let dateArr = date.split("/");
      for (let i = 0; i < dateArr.length; i++) {
        dateArr[i] = Number(dateArr[i]);
      }
      that.calendar.curdate = dateArr;
      let monthFirstDay = `${dateArr[0]}-${dateArr[1]}-01`;
      calendar.monthlen = that.getMonthLen(dateArr);
      calendar.prevmonthlen = that.getMonthLen(dateArr, "prev");
      calendar.nextmonthlen = that.getMonthLen(dateArr, "next");
      calendar.monthfirstdayweek = that.getDayWeek(monthFirstDay);
      let pmArr = [];
      pmArr = [...Array(calendar.prevmonthlen + 2).keys()].slice(1);
      let cmArr = [];
      cmArr = [...Array(calendar.monthlen + 1).keys()].slice(1);
      let nmArr = [];
      nmArr = [...Array(calendar.nextmonthlen + 1).keys()].slice(1);
      let pArr = pmArr.slice(-calendar.monthfirstdayweek, -1);
      let nstart = 42 - calendar.monthlen - calendar.monthfirstdayweek + 1;
      let nArr = nmArr.splice(0, nstart);
      let cArr = [];
      cArr = [...pArr, ...cmArr, ...nArr];
      calendar.contents.length = 42;
      for (let i = 0; i < cArr.length; i++) {
        let str = `${dateArr[0]}-${dateArr[1]}-${cArr[i]}`;
        if (i < calendar.monthfirstdayweek - 1 && dateArr[1] !== 1) {
          str = `${dateArr[0]}-${dateArr[1] - 1}-${cArr[i]}`;
        } else if (i < calendar.monthfirstdayweek - 1 && dateArr[1] === 1) {
          str = `${dateArr[0] - 1}-12-${cArr[i]}`;
        } else if (
          i >= calendar.monthfirstdayweek - 1 + calendar.monthlen &&
          dateArr[1] === 12
        ) {
          str = `${dateArr[0] + 1}-1-${cArr[i]}`;
        } else if (
          i >= calendar.monthfirstdayweek - 1 + calendar.monthlen &&
          dateArr[1] !== 12
        ) {
          str = `${dateArr[0]}-${dateArr[1] + 1}-${cArr[i]}`;
        }
        calendar.contents[i] = {
          val: str,
          active: false
        };
        if (
          calendar.contents[i].val.indexOf(`${dateArr[0]}-${dateArr[1]}`) > -1
        ) {
          calendar.contents[i].active = true;
        }
      }
      calendar.cArr = cArr;
      // that.addEventList();
      // that.addFestival();
      that.addMessage();
      that.$emit("calendarData", that.calendar);
    },
    getMonthLen(val, str) {
      let defval = new Date().toLocaleDateString().split("/");
      val ? val : (val = defval);
      let year = Number(val[0]);
      let month = Number(val[1]);
      if (str === "next") {
        if (month === 12) {
          year++;
        } else {
          month++;
        }
      } else if (str === "prev") {
        if (month === 1) {
          year--;
        } else {
          month--;
        }
      }
      let len = 31;
      if (month === 2) {
        let t1 = year % 4 === 0 && year % 100 !== 0;
        let t2 = year % 400 === 0;
        t1 || t2 ? (len = 29) : (len = 28);
      } else if (month === 4 || month === 6 || month === 9 || month === 11) {
        len = 30;
      }
      return len;
    },
    addMessage() {
      let that = this;
      let contents = that.calendar.contents;
      let festival = that.calendar.festival;
      let eventList = that.calendar.evenList;
      let dateArr = that.calendar.curdate;
       // 母亲节
      if(dateArr[1]===5){
        let str=15-that.getDayWeek(`${dateArr[0]}-5-1`)
        festival.push({date:`5-${str}`,festival:[{name:'母亲节'}]})
        }
        // 父亲节
        if(dateArr[1]===6){
          let str=22-that.getDayWeek(`${dateArr[0]}-6-1`)
        festival.push({date:`6-${str}`,festival:[{name:'父亲节'}]})
        }
        // 感恩节
        if(dateArr[1]===11){
          let str=33-that.getDayWeek(`${dateArr[0]}-11-1`)
        festival.push({date:`11-${str}`,festival:[{name:'感恩节'}]})
        }
        // 清明节
        if(dateArr[1]===4){
         let str=5
         if(that.isLeapYear(dateArr[0])||that.isLeapYear(dateArr[0]-1)){
           str=4
         }
        festival.push({date:`4-${str}`,festival:[{name:'清明节',vacation:true}]})
        }
      for (let x in contents) {
        for (let y in festival) {
          festival[y].tempdate = `${dateArr[0]}-${festival[y].date}`;
          if (contents[x].val === festival[y].tempdate) {
            contents[x].festival = festival[y].festival;
          }
        }
        for (let z in eventList) {
          if (eventList[z].date === contents[x].val) {
            contents[x].msg = eventList[z].msg;
          }
        }
      }
    },
    getDayWeek(val) {
      val ? val : (val = new Date().toLocaleDateString());
      let week = new Date(val).getDay();
      if (week === 0) {
        week = 7;
      }
      return week;
    },
    isLeapYear(year){
     let t1 = year % 4 === 0 && year % 100 !== 0;
     let t2 = year % 400 === 0; 
     if(t1||t2){
       return true
     }else{
       return false
     }
    },
    addEvent(obj) {
      let that = this;
      let evenList = that.calendar.evenList;
      if (obj instanceof Array) {
        evenList.concat(obj);
      } else {
        evenList.push(obj);
      }
    }
  },
  mounted() {
    this.init();
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.calendar {
  max-width: 100vw;
  display: flex;
  justify-content: flex-start;
}
.calendar-title,
.calendar-content {
  display: flex;
  justify-content: center;
  align-items: center;
  flex-flow: column wrap;
}
.calendar-title {
  width: 20%;
  height: 560px;
  max-height: 560px;
}
.calendar-content {
  width: 80%;
  max-height: 560px;
}
.calendar .calendar-title span,
.calendar .calendar-content .day-block {
  min-height: 80px;
  min-width: 140px;
  width: 16.667%;
  position: relative;
    display: flex;
  justify-content: center;
  align-items: center;
}
.calendar .calendar-title span{
    width: 100%;
    font-size: 20px;
    font-weight: bold;
}
.calendar .calendar-title span.weekend{
  color:#E3625D;
}
.calendar .calendar-title span:after,
.calendar .calendar-content .day-block:after{
  content:'';
  border-bottom: 1px dashed #ccc;
  width: 100%;
  height: 1px;
  position: absolute;
  bottom: 0;
  left: 0;
}
.calendar .calendar-content .day-block {
  color: #999;
   justify-content: flex-start;
}
.calendar-content .day-block .event,
.calendar-content .day-block .festival{
  padding-left: 10px;
}
.calendar-content .day-block .event{
  color:#F6D557;
}
.calendar-content .day-block .festival{
  color:#E3625D;
}
.calendar .calendar-content .active {
  color: #333;
}
</style>
