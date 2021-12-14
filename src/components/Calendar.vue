<template>
  <div class="hello">
    <!-- create event modal -->
    <div class="modal-backdrop" v-show="isModalVisible" @close="closeModal">
      <div class="modal">
        <header class="modal-header">
          <slot name="header"> Add an event </slot>
          <button type="button" class="btn-close" @click="closeModal">x</button>
        </header>
        <section class="modal-body">
          <slot name="body"> This is the default body! </slot>
        </section>

        <footer class="modal-footer">
          <slot name="footer"> </slot>
          <button type="button" class="btn-green" @click="closeModal">
            Close Modal
          </button>
        </footer>
      </div>
    </div>

    <div class="header">
      <h3 class="monthDisplay">{{ thisMonth }} {{ currentYear }}</h3>

      <button class="headerButton" @click="goToToday">Today</button>
      <button class="headerButton" @click="decreaseMonth">&lt;</button>
      <button class="headerButton" @click="increaseMonth">&gt;</button>

      <!-- <button class="headerButton">Month</button> -->
    </div>
    <div class="mainContent">
      <div class="sideContent">
        <h2>Upcoming Events</h2>
      </div>
      <div class="calendarDiv">
        <div class="weekDays">
          <span class="day" v-for="day in weekdays" :key="day">
            <h4>{{ day }}</h4>
          </span>
        </div>

        <!-- ------------------------- -->
        <div class="week" v-for="week in weeks" :key="week">
          <div
            class="calendarDay"
            :class="{ today: day.isToday }"
            v-for="day in week"
            :key="day"
          >
            <a class="dayLink" @click="showModal"> {{ day.label }} </a>
          </div>
        </div>
        <!-- ------------------------- -->
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "Calendar",
  data() {
    return {
      isModalVisible: false,
      date: new Date(),
      currentMonth: new Date().getMonth(),
      currentYear: new Date().getFullYear(),
      monthNames: [
        "January",
        "February",
        "March",
        "April",
        "May",
        "June",
        "July",
        "August",
        "September",
        "October",
        "November",
        "December",
      ],
      shortMonthName: [
        "Jan",
        "Feb",
        "Mar",
        "Apr",
        "May",
        "Jun",
        "Jul",
        "Aug",
        "Sep",
        "Oct",
        "Nov",
        "Dec",
      ],
      weekdays: ["Sun", "Mon", "Tue", "Wed", "Thu", "Fri", "Sat"],
    };
  },
  methods: {
    decreaseMonth() {
      if (this.currentMonth == 0) {
        this.currentMonth = 11;
        this.currentYear = this.currentYear - 1;
      } else {
        this.currentMonth = this.currentMonth - 1;
      }
    },
    increaseMonth() {
      if (this.currentMonth == 11) {
        this.currentMonth = 0;
        this.currentYear = this.currentYear + 1;
      } else {
        this.currentMonth = this.currentMonth + 1;
      }
    },
    goToToday() {
      this.date = new Date();
      this.currentMonth = this.date.getMonth();
      this.currentYear = this.date.getFullYear();
    },
    showModal() {
      this.isModalVisible = true;
    },
    closeModal() {
      this.isModalVisible = false;
    },
    close() {
      this.$emit("close");
    },
  },
  computed: {
    thisMonth() {
      return this.monthNames[this.currentMonth];
    },
    lastMonth() {
      return this.monthNames[this.currentMonth - 1];
    },
    nextMonth() {
      if (this.currentMonth < 11) {
        return this.monthNames[this.currentMonth + 1];
      } else {
        return this.monthNames[0];
      }
    },
    datesInMonth() {
      var month = this.currentMonth + 1;
      var year = this.currentYear;
      var monthNumbers = [];
      var daysInMonth = new Date(year, month, 0).getDate();
      for (let i = 0; i < daysInMonth; i++) {
        monthNumbers.push(i + 1);
      }
      return monthNumbers[monthNumbers.length - 1];
    },
    firstDayInMonth() {
      var month = this.currentMonth;
      var year = this.currentYear;
      var firstDate = new Date(year, month, 1);
      // var firstDayIndex = firstDate.getDay();
      // var fristWeekday = this.weekdays[firstDayIndex];
      // console.log(firstDayIndex);
      // console.log(fristWeekday);

      return firstDate;
    },

    firstWeekdayInMonth() {
      var column =
        new Date(this.currentYear, this.currentMonth, 1).getDay() + 1;
      return column;
    },
    /////////////////////////////////////
    weeks() {
      var today = new Date();
      var year = today.getFullYear();
      var month = today.getMonth + 1;
      //var day = today.getDate();
      const weeks = [];
      let monthStarted = false,
        monthEnded = false;
      let monthDay = 0;
      //cycle through each week of the month, up to 6 total
      for (let w = 1; w <= 6 && !monthEnded; w++) {
        //cycle through each weekday
        const week = [];
        for (let d = 1; d <= 7; d++) {
          //need to know when to start counting actual month days
          if (!monthStarted && d >= this.firstWeekdayInMonth) {
            //initialize day counter
            monthDay = 1;
            //and flag we're tracking actual month days
            monthStarted = true;
            //still in the middle of the month
          } else if (monthStarted && !monthEnded) {
            //increment day counter
            monthDay += 1;
          }
          //append day info for the current week
          //note: this might or might not be an actual month day
          week.push({
            label: monthDay ? monthDay.toString() : "",
            number: monthDay,
            weekdayNumber: d,
            weekNumber: w,
            beforeMonth: !monthStarted,
            afterMonth: monthEnded,
            inMonth: monthStarted && !monthEnded,
            isToday:
              monthDay === today &&
              this.currentMonth + 1 === month &&
              this.currentYear === year,
            isFirstDay: monthDay === 1,
            isLastDay: monthDay === this.datesInMonth,
          });

          //triger end of month on last day
          if (monthStarted && !monthEnded && monthDay >= this.datesInMonth) {
            monthDay = 0;
            monthEnded = true;
          }
        }
        //apend week info for the month
        weeks.push(week);
      }
      console.log(weeks);
      return weeks;
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.header {
  display: flex;
  justify-content: right;
  border-bottom: 1px solid black;
}
.headerButton {
  height: 40px;
  padding: 5px;
  margin: 5px;
  font-size: 90%;
  border-radius: 6px;
  border: 2px solid #e7e7e7;
  cursor: pointer;
}
.mainContent {
  display: grid;
  grid-template-areas: "side calendar calendar calendar";
}
.calendarDiv {
  grid-area: calendar;
}
.sideContent {
  grid-area: side;
}
.monthDisplay {
  border-bottom: none;
}
.weekDays {
  display: flex;
  height: 50px;
  border-bottom: 1px solid black;
}
.row {
  display: flex;
  align-items: center;
  justify-content: center;
}
.grid {
  border: 1px solid black;
  width: 100%;
}

/*-----------  */
.week {
  display: flex;
}

.day {
  width: 14.2857%;
  height: 50px;
  display: flex;
  justify-content: center;
  align-items: center;
  border: solid 1px #aaaaaa;
}

.calendarDay {
  width: 14.2857%;
  height: 100px;
  display: flex;
  justify-content: center;
  align-items: center;
  border: solid 1px #aaaaaa;
}

/* ------------- */
.calendarDay a {
  cursor: pointer;
}

ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}

/* modal style */
.modal-backdrop {
  position: fixed;
  top: 0;
  bottom: 0;
  left: 0;
  right: 0;
  background-color: rgba(0, 0, 0, 0.3);
  display: flex;
  justify-content: center;
  align-items: center;
}

.modal {
  background: #ffffff;
  box-shadow: 2px 2px 20px 1px;
  overflow-x: auto;
  display: flex;
  flex-direction: column;
}

.modal-header,
.modal-footer {
  padding: 15px;
  display: flex;
}

.modal-header {
  position: relative;
  border-bottom: 1px solid #eeeeee;
  color: #4aae9b;
  justify-content: space-between;
}

.modal-footer {
  border-top: 1px solid #eeeeee;
  flex-direction: column;
  justify-content: flex-end;
}

.modal-body {
  position: relative;
  padding: 20px 10px;
}

.btn-close {
  position: absolute;
  top: 0;
  right: 0;
  border: none;
  font-size: 20px;
  padding: 10px;
  cursor: pointer;
  font-weight: bold;
  color: #4aae9b;
  background: transparent;
}

.btn-green {
  color: white;
  background: #4aae9b;
  border: 1px solid #4aae9b;
  border-radius: 2px;
  cursor: pointer;
}
</style>
