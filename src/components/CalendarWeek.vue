<template>
  <tr>
    <CalendarDate
      v-for="(date, index) in week"
      :key="index"
      :date="date"
      :is-today="isToday === date"
      :is-selected="selectedDatesIncludes(date)"
      @selectDate="selectDate"
    />
  </tr>
</template>

<script>
import CalendarDate from './CalendarDate';
export default {
  components: {
    CalendarDate
  },
  props: {
    week: {
      type: Array,
      default: () => []
    },
    isToday: {
      type: Number,
      default: 0
    },
    allSelected: {
      type: Array,
      default: () => []
    }
  },
  methods: {
    selectDate(date) {
      this.$emit('selectDate', date);
    },
    selectedDatesIncludes(val) {
      return this.allSelected.map(date => date.date).includes(val);
    }
  }
}
</script>

<style scoped>
tr {
  color: #614a52;
}
</style>