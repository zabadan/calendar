<template>
  <div id="app" class="d-flex flex-center">
    <calendar-schedule :checkedDay="checkedDay" />
  </div>
</template>

<script>
export default {
  name: 'App',
  components: {
    CalendarSchedule: () => import('@/components/CalendarSchedule')
  },
  data () {
    return {
      checkedDay: {
        "mo": [
          {
            "bt": 240,
            "et": 779
          }
        ],
        "tu": [
        ],
        "we": [
        ],
        "th": [
          {
          "bt": 240,
                "et": 779
          },
          {
          "bt": 1140,
                "et": 1319
          }
        ],
        "fr": [
          {
          "bt": 660,
                "et": 1019
          }
        ],
        "sa": [
              {
                "bt": 0,
                "et": 1439
              }
            ],
        "su": []
        }
    }
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
