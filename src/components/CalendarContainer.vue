<template>
  <div class="calendar">
    <div class="calendar-header">
      <h2>{{ currentMonth + ' ' + currentYear }}</h2>
      <div class="calendar-nav">
        <span class="chevron chevron-left" @click="subtractMonth">&larr;</span>
        <span class="btn-today" @click="resetDateContext">Today</span>
        <span class="chevron chevron-right" @click="addMonth">&rarr;</span>
      </div>
    </div>

    <div class="calendar-body">
      <div class="weekday" v-for="day in weekdays" :key="day.id">{{ day }}</div>
      <div class="no-date" v-for="index in firstDayOfMonth" :key="index.id">{{ index }}</div>
      <div class="date" v-for="date in daysInMonth" :key="date.id" :class="{ 'today': date == initialDate && currentMonth == initialMonth && currentYear == initialYear }">
        <h3>{{ date }}</h3>
      </div>
    </div>
  </div>
</template>

<script>
  import { format, getDaysInMonth, getDay, startOfMonth, addMonths } from 'date-fns'
  import axios from 'axios'

  const now = new Date()
  const url = 'https://scheduler-challenge.herokuapp.com/schedule'

  export default {
    name: 'CalendarContainer',
    data() {
      return {
        weekdays: ['Sun', 'Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat'],
        today: now,
        dateContext: now,
        events: null
      }
    },
    computed: {
      initialYear: function () {
        return format(this.today, 'YYYY')
      },
      initialMonth: function () {
        return format(this.today, 'MMMM')
      },
      initialDate: function () {
        return format(this.today, 'D')
      },
      currentYear: function () {
        return format(this.dateContext, 'YYYY')
      },
      currentMonth: function () {
        return format(this.dateContext, 'MMMM')
      },
      currentDate: function () {
        return format(this.dateContext, 'D')
      },
      daysInMonth: function () {
        return getDaysInMonth(this.dateContext)
      },
      firstDayOfMonth: function () {
        return getDay(startOfMonth(this.dateContext))
      }
    },
    methods: {
      addMonth: function () {
        this.dateContext = addMonths(this.dateContext, 1)
      },
      subtractMonth: function () {
        this.dateContext = addMonths(this.dateContext, -1)
      },
      resetDateContext: function () {
        this.dateContext = this.today
      }
    },
    mounted() {
      axios
        .get(url)
        .then(response => {
          this.events = response.data
        })
    }
  }
</script>

<style scoped lang="scss">
  @import '../styles/calendar.scss'
</style>
