<!-- 日历组件 -->
<template>
  <div class="calendar-comp" id="calendar">
    <!-- 日历切换头 -->
    <div class="calendar-header-change">
      <!-- 左侧箭头 -->
      <div class="left-arrow">
        <span @click="preDate" v-show="isPrev">{{ leftArrow }}</span>
      </div>
      
      <!-- 日期展示区域 -->
      <div class="calendar-date-show">
        <span @click="startSelectArea">{{ showDate }}</span>
        <img src="http://img4.imgtn.bdimg.com/it/u=1271566325,3634786947&fm=26&gp=0.jpg" @click="resetDate" alt="">

        <div class="calendar-select-yearArea" v-show="dateListArea">
          <!-- 选择你年份 -->
          <div class="calendar-year-list" v-if="yearList">
            <span 
              v-for="(item, index) in dateAreaList" 
              :key="index" 
              @click="selectYear(item.yearDate,index)"
              :class="activeYear == item.yearDate ? 'activeYear' : ''">{{ item.yearDate }}</span>
          </div>

          <!-- 选择你月份 -->
          <div class="calendar-month-list" v-else-if="!yearList">
            <div class="goPrev" @click="goYear">{{ leftArrow }}</div>
            <span 
              v-for="(item, index) in dateAreaMonth" 
              :key="index" 
              @click="selectMonth(item)"
              :class="activeMonth == item ? 'activeMonth' : ''">{{ item }}月</span>
          </div>
          <div class="calendar-button-group">
            <!-- 取消 -->
            <div class="cancel" @click="cancel">取消</div>
            <!-- 确认 -->
            <div class="sure" @click='sureDate'>确认</div>
          </div>
        </div>
      </div>

      <!-- 右侧箭头 -->
      <div class="right-arrow">
        <span @click="nextDate" v-show="isNext">{{ rightArrow }}</span>
      </div>
    </div>

    <!-- 日历正文内容 -->
    <div class="calendar-content">
      <div class="calendar-content-week">
        <span 
          v-for="(item,index) in weeks"
          :key="index"
        >
          {{ item }}
          <i :class="{'activeweek':index == weekActive}"/>
        </span>
      </div>

      <div class="calendar-content-day">
        <span 
          v-for="(item,index) in dateList" 
          :key="index"
          :class="{'noActive': item.dateType == 0 || item.minDate == 0 || item.dateType == 2,'activeDate': item.showDate == parseInt(startDay) && item.dateType == 1}"
          @click="selectDate(item)">{{ item.showDate }}</span>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    // 开始日期
    startDate: {
      type: String,
      default: ""
    },
    // 结束日期
    endDate: {
      type: String,
      default: ""
    }
  },
  data() {
    return {
      weeks: ["日", "一", "二", "三", "四", "五", "六"], //星期数组
      startYear: "", //开始年
      startMonth: "", //开始月份
      startDay: "", // 开始日期
      endYear: "", //结束年
      endMonth: "", //结束月
      endDay: "", //结束日期
      dateList: [],
      showDate: "",
      weekActive: 9,
      yearList: true,     //控是否显示年的列表
      dateListArea: false,      //控制日期选择列表的显示与隐藏
      dateAreaList: [],
      dateAreaMonth: [],
      leftArrow: '<',
      rightArrow: '>',
      activeYear: '',
      activeMonth: ''
    };
  },

  components: {},

  computed: {
    isPrev() {
      let resultEnd = this.startDate.split("-");
      let showDate = this.showDate.split("-");
      return showDate[0] == resultEnd[0] && showDate[1] == resultEnd[1]
        ? false
        : true;
    },
    isNext() {
      let resultEnd = this.endDate.split("-");
      let showDate = this.showDate.split("-");
      return showDate[0] == resultEnd[0] && showDate[1] == resultEnd[1]
        ? false
        : true;
    }
  },

  mounted() {
    if (this.startDate) {
      let result = this.startDate.split("-");
      this.startYear = result[0];
      this.startMonth = result[1];
      this.startDay = result[2];
      this.activeYear = result[0];
      this.activeMonth = result[1];
    } else {
      let today = new Date();
      this.startYear = today.getFullYear();
      this.startMonth = today.getMonth() + 1;
      this.startDay = today.getDate();
      this.activeYear = today.getFullYear();
      this.activeMonth = today.getMonth() + 1;
    }
    
    this.setEndDate();

    if(new Date(this.endYear, this.endMonth, this.endDay) - new Date(this.startYear, this.startMonth, this.startDay) < 0) {
      this.errorToast("结束日期不能小于开始日期，请重新设置")
      return;
    }

    this.setDateObj();
    this.startDateHindle(this.startDate);
  },

  methods: {
    // 开始日期的处理
    startDateHindle(dateParams = "", monthType = true) {
      this.dateList = [];
      // 如果传递过来的数据为空，就获取当前日期
      if (!dateParams) {
        dateParams =
          this.startYear + "-" + this.startMonth + "-" + this.startDay;
      }

      // 父组件传递过来数据
      let resultDate = dateParams.split("-");
      this.showDate = resultDate[0] + "-" + resultDate[1];

      // 设置星期的高亮显示
      this.setWeekHeight(
        parseInt(resultDate[0]),
        parseInt(resultDate[1]) - 1,
        parseInt(resultDate[2])
      );

      let firstDay = new Date(
        parseInt(resultDate[0]),
        parseInt(resultDate[1]) - 1,
        1
      ); //当前月的第一天
      let firstDayWeekDay = firstDay.getDay(); //当前月的第一天是星期几

      firstDayWeekDay === 0 ? (firstDayWeekDay = 7) : ""; //星期是1,2,3,4,5,6,0。所以星期天为0；

      let lastDayOfLastMonth = new Date(
        parseInt(resultDate[0]),
        parseInt(resultDate[1]) - 1,
        0
      ); //上一个月的最后天
      let lastDateOfLastMonth = lastDayOfLastMonth.getDate();

      // 日历第一行要显示多少上一个月的日期
      let preMonthDayCount = firstDayWeekDay;

      let lastDay = new Date(
        parseInt(resultDate[0]),
        parseInt(resultDate[1]),
        0
      ); //当前月的最后一天
      let lastDate = lastDay.getDate();

      for (let i = 0; i < 7 * 6; i++) {
        let date = i + 1 - preMonthDayCount;
        let showDate = date;
        let thisMonth = parseInt(resultDate[1]);
        let dateType = 1; //当前日期类别，0：上一个月   1：本月       2：下一个月
        let minDate = 1;

        // 上一个月
        if (date <= 0) {
          thisMonth = parseInt(resultDate[1]) - 1;
          showDate = lastDateOfLastMonth + date;
          minDate = 0;
          dateType = 0;
        } else if (date > lastDate) {
          // 下一个月
          thisMonth = parseInt(resultDate[1]) + 1;
          showDate = showDate - lastDate;
          dateType = 2;
        } else if (date > 0 && date < parseInt(resultDate[2]) && monthType) {
          minDate = 0;
        }

        // 上一个月的日期全部置灰且不可点击
        if (date > 0 && date <= lastDate && monthType) {
          if (
            (parseInt(resultDate[0]) == this.startYear &&
              parseInt(resultDate[1]) < this.startMonth) ||
            parseInt(resultDate[0]) < this.startYear
          ) {
            dateType = 0;
            this.weekActive = 9;
          }
        }

        //最小日起限制
        if (
          this.endYear + "-" + this.endMonth == this.showDate &&
          date > parseInt(resultDate[2]) &&
          date <= lastDate
        ) {
          dateType = 0;
        }

        if (thisMonth === 0) thisMonth = 12;
        if (thisMonth === 13) thisMonth = 1;

        this.dateList.push({
          month: thisMonth,
          data: date,
          showDate: showDate,
          dateType: dateType,
          minDate: minDate
        });
      }
    },
    // 设置星期的高亮显示
    setWeekHeight(year, month, day) {
      let result = new Date(year, month, day);
      this.weekActive = result.getDay();
    },
    // 选择日期
    selectDate(data) {
      if (data.dateType == 1 && data.minDate != 0) {
        this.startDay = data.showDate;
        let dateArr = this.showDate.split("-");
        this.setWeekHeight(
          parseInt(dateArr[0]),
          parseInt(dateArr[1]) - 1,
          data.showDate
        );
      }
      this.$emit("getDateVal",this.showDate + '-' + data.showDate)
    },
    // 日期处理：或加或减。typeMethod：add加   minus：减
    dateHandle(typeMethod) {
      let resultDate = this.showDate.split("-");
      let typemoth;

      let year = parseInt(resultDate[0]); //年
      let month = parseInt(resultDate[1]); //月

      if (typeMethod == "add") {
        month += 1;
        if (month == 13) {
          month = 1;
          year += 1;
        }
        this.showDate = year + "-" + month;
      } else if (typeMethod == "minus") {
        month -= 1;
        if (month == 0) {
          month = 12;
          year -= 1;
        }
        this.showDate = year + "-" + month;
      }

      if (
        (year == this.startYear && month > this.startMonth) ||
        year > this.startYear
      ) {
        typemoth = false;
      } else if (
        (year == this.startYear && month <= this.startMonth) ||
        year < this.startYear
      ) {
        typemoth = true;
      }

      this.startDateHindle(this.showDate + "-" + this.startDay, typemoth);
    },
    // 上一个日期
    preDate() {
      this.dateHandle("minus");
    },
    // 下一个日期
    nextDate() {
      this.dateHandle("add");
    },
    // 设置结束日期
    setEndDate() {
      let result = this.endDate
      if(result) {
        result = this.endDate.split("-");
        this.endYear = parseInt(result[0]);
      }else{
        result = [this.startYear + "", this.startMonth + "", this.startDay + ""];
        this.endYear = parseInt(result[0]) + 50;
      }
      this.endMonth = result[1];
      this.endDay = result[2];
    },
    // 重置
    resetDate() {
      this.startDateHindle(this.startDate);
    },
    // 选择年
    selectYear(val,idx) {
      this.activeYear = val;
      this.yearList = false
      this.dateAreaMonth = this.dateAreaList[idx]['month'];
    },
    //选择月
    selectMonth(val) {
      this.activeMonth = val;
    },
    // 开始选择日期
    startSelectArea() {
      this.dateListArea = true;
    } ,
    // 取消
    cancel() {
      this.dateListArea = false;
      this.yearList = true;
    },
    // 确认
    sureDate() {
      this.showDate = this.activeYear + '-' + this.activeMonth;
      let day = document.getElementsByClassName("activeDate")[0].innerText;
      if(this.activeYear == this.startYear && this.activeMonth == this.startMonth) {
        let result = this.activeYear + '-' + this.activeMonth + '-' + day;
        this.startDateHindle(result)
      }else if(this.activeYear == this.endYear && this.activeMonth == this.endMonth){
        let result = this.activeYear + '-' + this.activeMonth + '-' + day;
        this.startDateHindle(result)
      }else{
        let result = this.activeYear + '-' + this.activeMonth + '-' + day;
        this.startDateHindle(result)
      }
      this.$emit("getDateVal",this.activeYear + '-' + this.activeMonth + '-' + day)
      this.dateListArea = false;
      this.yearList = true;
    },
    // 整理日期，将年和月一一对应
    setDateObj() {
      this.dateAreaList = [];
      // 开始年与结束年相等
      if(this.startYear == this.endYear) {
        let date = {
          'year': this.startYear,
          'month': parseInt(this.endMonth) - parseInt(this.startMonth) + 1
        }

        this.dateAreaList.push(date)

      } else if(this.startYear < this.endYear) {
        // if(parseInt(this.startYear) - parseInt(this.endYear) == 1) {

        // }
        for(let year = parseInt(this.startYear); year <= parseInt(this.endYear); year++) {
          let dateObj = {
            yearDate: year 
          }
          if(year == parseInt(this.startYear)) {
            let rsult = this.startMonthHandle();
            dateObj["month"] = rsult
            this.dateAreaList.push(dateObj)
          }else if(year == parseInt(this.endYear)) {
            let rsult = this.endMonthHandle();
            dateObj["month"] = rsult
            this.dateAreaList.push(dateObj)
          }else{
            let rsult = this.normalMonthHandle();
            dateObj["month"] = rsult
            this.dateAreaList.push(dateObj)
          }
        }
      }
    },
    // 开始日期月份处理
    startMonthHandle() {
      let monthArr = [];
      for(let i =  parseInt(this.startMonth); i <= 12; i++) {
        monthArr.push(i);
      }
      return monthArr;
    },
    // 结束日期月份处理
    endMonthHandle() {
      let monthArr = [];
      for(let i =  1; i <= parseInt(this.endMonth); i++) {
        monthArr.push(i);
      }
      return monthArr;
    },
    normalMonthHandle() {
      let monthArr = [];
      for(let i =  1; i <= 12; i++) {
        monthArr.push(i);
      }
      return monthArr;
    },
    // 错误提示信息
    errorToast(msg) {
      this.$emit('errorMag',msg)
    },
    goYear() {
      this.yearList = true
    }
  }
};
</script>
<style scoped lang="scss">
.calendar-comp {
  width: 100%;
  height: auto;
  .calendar-header-change {
    width: 100%;
    height: 45px;
    display: flex;
    justify-content: space-between;
    align-items: center;
    .left-arrow,
    .right-arrow {
      width: 45px;
      height: 100%;
      display: flex;
      justify-content: center;
      align-items: center;
    }
    .calendar-date-show {
      flex: 1;
      height: 100%;
      display: flex;
      justify-content: center;
      align-items: center;
      font-size: 15px;
      color: #333;
      position: relative;
      img{
        margin-left: 8px;
        width: 15px;
        height: 15px;
      }
      .calendar-select-yearArea{
        position: absolute;
        width: 200px;
        height: 180px;
        z-index: 100;
        top: 40px;
        background: #ffffff;
        box-shadow: 0 0 8px 1px rgb(226, 224, 224);
        border-radius: 5px;
        display: flex;
        flex-direction: column;
        justify-content: space-between;
        align-items: center;
        overflow: hidden;
        .calendar-year-list,.calendar-month-list{
          width: 100%;
          flex: 1;
          overflow: scroll;
          overflow-x: hidden;
          overflow-y: auto;
          display: flex;
          flex-wrap: wrap;
          position: relative;
          span{
            display: inline-block;
            width: 50%;
            height: 30px;
            line-height: 30px;
            color: #333;
            font-size: 15px;
            text-align: center;
          }
          .activeYear,.activeMonth{
            color:#66abff;
          }
          .goPrev{
            width: 15px;
            height: 15px;
            border-radius: 50%;
            box-shadow: 0 0 8px 1px rgb(226, 224, 224);
            position: absolute;
            left: 10px;
            top: 10px;
            z-index: 100;
            text-align: center;
            line-height: 12px;
            font-size: 10px;
          }
        }
        .calendar-button-group{
          width: 100%;
          height: 40px;
          background: #66abff;
          display: flex;
          justify-content: space-between;
          align-items: center;
          .cancel,.sure{
            width: 50%;
            height: 100%;
            color:#ffffff;
            text-align: center;
            line-height: 40px;
            font-size: 15px;
          }
        }
      }
    }
  }
  .calendar-content {
    width: 100%;
    height: auto;
    .calendar-content-week {
      width: 100%;
      height: 45px;
      display: flex;
      justify-content: space-between;
      align-items: center;
      span {
        display: inline-block;
        width: 14.28%;
        height: 100%;
        display: flex;
        justify-content: center;
        align-items: center;
        color: #333;
        font-size: 15px;
        position: relative;
        i {
          display: inline-block;
          width: 100%;
          height: 2px;
          background: #ffffff;
          position: absolute;
          left: 0;
          bottom: 0;
          z-index: 10;
          border-bottom: 1px solid #ecf1f8;
        }
        .activeweek {
          background: #66abff;
        }
      }
    }
    .calendar-content-day {
      width: 100%;
      height: auto;
      display: flex;
      flex-wrap: wrap;
      align-items: center;
      span {
        display: inline-block;
        width: 14.28%;
        height: 45px;
        display: flex;
        justify-content: center;
        align-items: center;
        color: #333;
        font-size: 15px;
        border-radius: 6px;
      }
      .noActive {
        color: #999999;
      }
      .activeDate {
        color: #ffffff;
        background: linear-gradient(to right, #abdcff, #66abff);
      }
    }
  }
}
</style>