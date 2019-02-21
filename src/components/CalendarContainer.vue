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
      <div class="no-date" v-for="index in startDayOfMonth" :key="index.id">{{ index }}</div>
      <div class="date" v-for="date in daysInMonth" :key="date.id" :class="{ 'today': date == initialDate && currentMonth == initialMonth && currentYear == initialYear }">
        <h3><span>{{ date }}</span></h3>
        <div class="events" v-if="getEventsAtDate(date)">
          <CalendarEvent v-for="eventInfo in getEventsAtDate(date)" :key="eventInfo.id" :eventInfo="eventInfo"></CalendarEvent>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
  import axios from 'axios'
  import parser from 'cron-parser'
  import { format, getDaysInMonth, startOfMonth, getDay, endOfMonth, addMonths } from 'date-fns'

  import CalendarEvent from './CalendarEvent.vue'

  const now = new Date()
  const url = 'https://scheduler-challenge.herokuapp.com/schedule'

  export default {
    name: 'CalendarContainer',
    components: {
      CalendarEvent
    },
    data() {
      return {
        weekdays: ['Sun', 'Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat'],
        today: now,
        dateContext: now,
        eventsData: []
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
      daysInMonth: function () {
        return getDaysInMonth(this.dateContext)
      },
      startDateOfMonth: function () {
        return startOfMonth(this.dateContext)
      },
      startDayOfMonth: function () {
        return getDay(this.startDateOfMonth)
      },
      endDateOfMonth: function () {
        return endOfMonth(this.dateContext)
      },
      eventsInMonth: function () {
        let eventsObj = {}

        for (let event of this.eventsData) {
          try {
            // cron parser does not support cron format in extended mode
            // so need to make sure string has five fields
            let cronString = event.attributes.cron
            let cronArray = cronString.split(' ')

            if (cronArray.length < 5) {
              for (let i = cronArray.length; i < 5; i++) {
                cronArray[i] = '*'
              }
              cronString = cronArray.join(' ')
            }

            let cronIterator = parser.parseExpression(cronString, {
              currentDate: this.startDateOfMonth,
              endDate: this.endDateOfMonth,
              iterator: true
            })
            let cronIteratorHasNext = true

            while (cronIteratorHasNext) {
              try {
                let nextObj = cronIterator.next()
                let date = format(nextObj.value, 'D')

                if (!(date in eventsObj)) {
                  eventsObj[date] = []
                }

                eventsObj[date].push({
                  name: event.attributes.name,
                  time: format(nextObj.value, 'h:mma'),
                  timestamp: format(nextObj.value, 'X'),
                  type: event.type,
                })

                cronIteratorHasNext = !nextObj.done
              } catch (e) {
                break;
              }
            }
          } catch (e) {
            break
          }
        }
        return eventsObj
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
      },
      getEventsAtDate: function (date) {
        // sort list of events by timestamp
        let eventsAtDate = this.eventsInMonth[date]
        if (Array.isArray(eventsAtDate)) {
          eventsAtDate.sort((a, b) => (a.timestamp > b.timestamp) ? 1 : -1)
        }
        return eventsAtDate
      },
      checkEventIsStarting: function () {
        let nowTimestamp = format(new Date(), 'X')
        let eventsToday = this.getEventsAtDate(this.initialDate)
        for (let event of eventsToday) {
          if (event.timestamp == nowTimestamp) {
            this.createEventNotification(event.name)
          }
        }
      },
      createEventNotification: function (eventName) {
        if (Notification.permission === 'granted') {
          new Notification('Your event is starting now!', { body: eventName })
        }
        // chrome does not implement the permission static property
        // so need to check for NOT 'denied' instead of 'default'
        else if (Notification.permission !== 'denied') {
          Notification.requestPermission(function (permission) {
            // need to make sure Chrome stores the info
            if(!('permission' in Notification)) {
              Notification.permission = permission
            }
            if (permission === 'granted') {
              new Notification('Your event is starting now!', { body: eventName })
            }
          })
        }
      }
    },
    mounted() {
      axios
        .get(url)
        .then(response => {
          this.eventsData = response.data.data
        })

      setInterval(this.checkEventIsStarting, 60000)
    }
  }
</script>

<style scoped lang="scss">
  @import '../styles/calendar.scss'
</style>
