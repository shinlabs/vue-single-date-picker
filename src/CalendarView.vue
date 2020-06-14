/* eslint-disable no-console */
<template>
  <div
    id="single-date-picker"
    class="single-date-picker__calendar-view"
  >
    <CalendarMonthHeader
      :year="year"
      :month="month"
      @toggleMonth="toggleMonth"
    />
    <CalendarMonth 
      :dates-per-week="datesPerWeek"
      :is-today="isToday"
      :all-selected="allSelected"
      @selectDate="selectDate"
    />
  </div>
</template>

<script>
import CalendarMonthHeader from './components/CalendarMonthHeader';
import CalendarMonth from './components/CalendarMonth';

const NUM_DAYS_IN_WEEK = 7;

export default {
  name: 'CalendarView',
  components: {
    CalendarMonthHeader,
    CalendarMonth
  },
  data() {
    return {
      year: 0,
      month: 0,
      todayDate: 0,
      todayYear: 0,
      todayMonth: 0,
      selectedDates: []
    }
  },
  computed: {
    numDays() {
      return new Date(this.year, this.month + 1, 0).getDate();
    },
    firstDay() {
      return new Date(this.year, this.month, 1).getDay();
    },
    lastDay() {
      return new Date(this.year, this.month + 1, 0).getDay();
    },
    numDaysInFirstWeek() {
      return NUM_DAYS_IN_WEEK - this.firstDay;
    },
    numDaysInLastWeek() {
      return this.lastDay + 1;
    },
    numWeeks() {
      const daysLeft = this.numDays - this.numDaysInFirstWeek - this.numDaysInLastWeek;
      return daysLeft / 7 + 2;
    },
    numFullWeeks() {
      return this.numWeeks - 2;
    },
    datesInFirstWeek() {
      return this.generateDatesInWeek(1, this.firstDay, this.numDaysInFirstWeek);
    },
    datesInLastWeek() {
      return this.generateDatesInWeek(this.numDays - this.numDaysInLastWeek + 1, 0, this.numDaysInLastWeek);
    },
    datesInMiddleWeeks() {
      const startDates = [];
      for (let i = 0; i < this.numFullWeeks; i += 1) {
        startDates[i] = this.datesInFirstWeek[6] + 1 + i*7;
      }
      return startDates.map(startDate => {
        return this.generateDatesInWeek(startDate, 0, 7);
      });
    },
    datesPerWeek() {
      return [
        this.datesInFirstWeek,
        ...this.datesInMiddleWeeks,
        this.datesInLastWeek
      ]
    },
    isCurrentMonth() {
      return this.todayMonth === this.month;
    },
    isCurrentYear() {
      return this.todayYear === this.year;
    },
    isToday() {
      return (this.isCurrentMonth && this.isCurrentYear) ? this.todayDate : 0;
    },
    allSelected() {
      if (this.selectedDates.length > 0) {
        let currentDates = this.selectedDates.filter(date => date.year === this.year && date.month === this.month);
        return currentDates.length > 0 ? currentDates : [];
      }
      return [];
    }
  },
  watch: {
    date(val) {
      this.setDate(val);
    }
  },
  created() {
    const date = new Date();

    this.year = date.getFullYear();
    this.month = date.getMonth();

    this.todayDate = date.getDate();
    this.todayYear = date.getFullYear();
    this.todayMonth = date.getMonth();

  },
  methods: {
    generateDatesInWeek(startDate, startDay, numDays) {
      const datesInWeek = new Array(7).fill(0);
      for (let i = 0; i < numDays; i+=1) {
        datesInWeek[startDay + i] = startDate + i;
      }
      return datesInWeek;
    },
    toggleMonth(direction) {
      let newMonth = this.month + Number(direction);
      let newYear = this.year;
      if (newMonth < 0) {
        newMonth = 11;
        newYear -= 1; 
      }
      if (newMonth > 11) {
        newMonth = 0;
        newYear += 1;
      }
      if (newYear >= 1970) {
        this.month = newMonth;
        this.year = newYear;
      }
    },
    selectDate(date) {
      if (date) {
        let selectDateIndex = this.selectedDates.findIndex(selectedDate => selectedDate.date === date);
        if (selectDateIndex < 0) {
          this.selectedDates.push({
            year: this.year,
            month: this.month,
            date
          });
        } else {
          this.selectedDates.splice(selectDateIndex, 1);
        }

        let selectedDatesForEvent = this.selectedDates.map(date => date.year + '-' + date.month + '-' + date.date);
        selectedDatesForEvent.sort((a, b) => new Date(a) - new Date(b));
        
        const customSelectDatesEvent = new CustomEvent('customSelectDates', {
          bubbles: true,
          composed: true, 
          detail: {
            selectedDates: selectedDatesForEvent
          }});
        this.$el.dispatchEvent(customSelectDatesEvent);
      }
    }
  }
}
</script>

<style lang="scss" scoped>
#single-date-picker {
  font-family: 'Lato', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}

.single-date-picker__calendar-view {
  max-width: 300px;
  background-color: white;
  border-radius: 10px;
  box-shadow: 0 0 10px 3px rgba(0, 0, 0, 0.1);
  color: #0d1e44;
}

@import url("https://fonts.googleapis.com/css?family=Lato:100,300,400&display=swap");
</style>