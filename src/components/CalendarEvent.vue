<template>
  <div class="calendar-event" :class="'event-' + eventInfo.type">
    <span>{{ eventInfo.time }} {{ eventInfo.name }}</span>
  </div>
</template>

<script>
  export default {
    name: 'CalendarEvent',
    props: [
      'eventInfo'
    ]
  }
</script>

<style scoped lang="scss">
  @import '../styles/event.scss'
</style>