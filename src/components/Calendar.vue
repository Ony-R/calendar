<template>
  <div class="hello">
    <!-- create event modal -->
    <div class="modal-backdrop" v-show="isModalVisible">
      <div class="modal-wrapper" @click="closeModal" >
        <div class="modal" @click.stop="">
          <header class="modal-header">
            <slot name="header"> Add an event </slot>
            <button type="button" class="btn-close" @click="closeModal">
              x
            </button>
          </header>
          <section class="modal-body">
            <slot name="body"> 
              <p style="font-weight: bold">{{ selectedDate }} </p>
              <div class="eventForm">
              <label>Event name: </label><input v-model="eventName"/>
              <br/>
              <label>Time (optional): </label><input v-model="eventTime"/>
              <br/>
              <label>Description (optional): </label><textarea v-model="eventDescription"></textarea>
              <p v-show="showEventError" class="error">Please enter an event name.</p>
              </div>
            </slot>
          </section>

          <footer class="modal-footer">
            <slot name="footer"> </slot>
            <button type="button" class="btn-green" @click="addEvent">
              Add event
            </button>
          </footer>
        </div>
      </div>
    </div>
    <!-- header -->
    <div class="header">
      <h3 class="monthDisplay">{{ thisMonth }} {{ currentYear }}</h3>

      <button class="headerButton" @click="goToToday">Today</button>
      <button class="headerButton" @click="decreaseMonth">&lt;</button>
      <button class="headerButton" @click="increaseMonth">&gt;</button>

      <!-- <button class="headerButton">Month</button> -->
    </div>
    <!-- main content -->
    <div class="mainContent">


      <!-- events/side content -->
      <div class="sideContent">
        <div class="select-dropdown">
          <select v-model="viewSelect">
            <option>Upcoming Events</option>
            <option>All Events</option>
          </select>
        </div>
        <div v-if="viewSelect == 'Upcoming Events'">
          <div class="eventDetails" v-for="event in eventsList" :key="event" >
            <div v-if="event.isCurrent">
              <h3>{{event.date}}</h3>
              <div style="font-weight: bold">{{event.name}}              
                <span v-if="event.showTime"> @ {{event.time}}  </span>    
              </div>  
              <p>{{event.description}}</p>
            </div>
          </div>
        </div>
        <div v-if="viewSelect == 'All Events'">
          <div class="eventDetails" v-for="event in eventsList" :key="event" >
              <h3>{{event.date}}</h3>
              <div style="font-weight: bold">{{event.name}}                 
                <span v-if="event.showTime"> @ {{event.time}}  </span>              
              </div>  
              <p>{{event.description}}</p>
          </div>
        </div>
      </div>




      <!-- calendar content -->
      <div class="calendarDiv">
        <div class="weekDays">
          <span class="day" v-for="day in weekdays" :key="day">
            <h4>{{ day }}</h4>
          </span>
        </div>

        <div class="week" v-for="week in weeks" :key="week">
          <div
            class="calendarDay"
            :class="{ today: day.isToday }"
            v-for="day in week"
            :key="day"
            @click="getCurrentDate"
          >
            <a class='dayLink' :class='{today: day.isToday}' @click="showModal"> {{ day.label }} </a>
            
            <!-- <p v-if=""></p>  make computed property -->
            <div v-for="event in eventsList" :key="event" >
              
              <p class="eventOnCalendar" v-show="event.dateString == day.dateString">{{event.name}}</p>
              
            </div>
          </div>
        </div>
      </div>
    </div>
    <!-- main content div -->
  </div>
  <!-- "hello" div -->
</template>

<script>
export default {
  name: "Calendar",
  data() {
    return {
      viewSelect: 'Upcoming Events',
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
      selectedDate: '',
      eventsList: [],
      eventName: '',
      eventDescription: '',
      eventTime: '',
      showEventError: false,
      isCurrent: false
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
      this.showEventError = false;
    },
    closeModal() {
      this.isModalVisible = false;
      this.eventName = '';
      this.eventDescription = '';
      this.eventTime = '';
    },
    getCurrentDate() {
      var month = this.monthNames[this.currentMonth];
      var day = event.target.parentElement.children[0].text;
      var year = this.currentYear;
      this.selectedDate = month + ' ' + day + ', ' + year;
    },
    addEvent() {
      var dayNumber = this.selectedDate.split(' ')[1].split(',')[0];
      var month = this.currentMonth + 1;
      var dateString = this.currentYear + '-' + month + '-' + dayNumber;
      var date = new Date();
      var newDate = date.setUTCHours(0,0,0,0);
      var calendarDate =  new Date(dateString).getTime();

      //check if the event happens before today
      if(newDate > calendarDate) {
        this.isCurrent = false;
      } else {
        this.isCurrent = true;
      } 
      
      if(this.selectedDate && this.eventName) {
        this.eventsList.push({
          date: this.selectedDate, 
          name: this.eventName, 
          description: this.eventDescription, 
          time: this.eventTime, 
          showTime: this.eventTime ? true : false,
          isCurrent: this.isCurrent ? true : false,
          dayNumber: dayNumber,
          dateString: dateString
          
          });
        this.isModalVisible = false;
        this.eventName = '';
        this.eventDescription = '';
        this.eventTime = '';
        this.showEventError = false;
      }
      else {
        this.showEventError = true;
      }    
      console.log(this.eventsList)
    }
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
    weeks() {
      var today = new Date();
      var year = today.getFullYear();
      var month = today.getMonth() + 1;
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
            monthDay: monthDay,
            label: monthDay ? monthDay.toString() : "",
            number: monthDay,
            weekdayNumber: d,
            weekNumber: w,
            beforeMonth: !monthStarted,
            afterMonth: monthEnded,
            inMonth: monthStarted && !monthEnded,
            isToday:
              monthDay === (today.getDate()) &&
              this.currentMonth + 1 === month &&
              this.currentYear === year,
            isFirstDay: monthDay === 1,
            isLastDay: monthDay === this.datesInMonth,
            dateString: this.currentYear + '-' + (this.currentMonth + 1) + '-' + monthDay,
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
      //console.log(weeks)
      return weeks;
    },
    filteredEvents() {
      //var filteredList = [];
      var events = this.eventsList;
      console.log(events)
 
      return events;
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

/*----------- calendar */
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
  text-align: center;
  /* display: flex;
  justify-content: center;
  align-items: center; */
  border: solid 1px #aaaaaa;
}

.calendarDay a {
  cursor: pointer;
}

.eventOnCalendar {
  
}

ul {
  list-style-type: none;
  padding: 0;
}
li {
  /* display: inline-block; */
  margin: 0 10px;
}
a {
  color: #42b983;
}

/* modal style */
.modal-backdrop {
  /* position: fixed;
  z-index: 99998;
  top: 0;
  bottom: 0;
  left: 0;
  right: 0;
  background-color: rgba(0, 0, 0, 0.3);
  display: flex;
  justify-content: center;
  align-items: center; */


  position: fixed;
  z-index: 9998;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, .5);
  display: table;
  transition: opacity .3s ease;
}

.modal-wrapper {
  display: table-cell;
  vertical-align: middle;
}
.modal {
  /* background: #ffffff;
  box-shadow: 2px 2px 20px 1px;
  overflow-x: auto;
  display: flex;
  flex-direction: column; */

  width: 300px;
  margin: 0px auto;
  padding: 20px 30px;
  background-color: #fff;
  border-radius: 2px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, .33);
  transition: all .3s ease;
  font-family: Helvetica, Arial, sans-serif;
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
.error {
  color: red;
}

.eventForm {
  margin: auto;
  width: 300px;
  text-align: left;
  clear: both;

}
.eventForm input {
  width: 85%;
  height: 20px;
  border-radius: 8px;
  clear: both;
  margin-top: 3px;
  margin-bottom: 5px;
}
.eventForm input:focus {
  outline: none;
}
.eventForm textarea {
  width: 85%;
  height: 50px;
  border-radius: 8px;
  margin-top: 3px;
  border: 2px solid black;
}
.eventForm textarea:focus {
  outline: none;
}
.today {
  font-weight: 500;
  color: grey;
  background-color: lightblue;
}

.select-dropdown,
.select-dropdown * {
  margin: 0;
  padding: 0;
  position: relative;
  box-sizing: border-box;
}
.select-dropdown {
  position: relative;
  background-color: #e4f2f5;
  border-radius: 4px;
}
.select-dropdown select {
  font-size: 1rem;
  font-weight: normal;
  width: 100%;
  padding: 8px 24px 8px 10px;
  border: none;
  background-color: transparent;
    -webkit-appearance: none;
    -moz-appearance: none;
  appearance: none;
}
.select-dropdown select:active, .select-dropdown select:focus {
  outline: none;
  box-shadow: none;
}
.select-dropdown:after {
  content: "";
  position: absolute;
  top: 50%;
  right: 8px;
  width: 0;
  height: 0;
  margin-top: -2px;
  border-top: 5px solid #aaa;
  border-right: 5px solid transparent;
  border-left: 5px solid transparent;
}

</style>
