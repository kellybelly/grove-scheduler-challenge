<template>
  <div id="app">
    <CalendarContainer></CalendarContainer>
  </div>
</template>

<script>
import CalendarContainer from './components/CalendarContainer.vue'

export default {
  name: 'app',
  components: {
    CalendarContainer
  }
}
</script>

<style lang="scss">
  @import './styles/base.scss'
</style>
