<template>
  <div>
    <h1 class="calendar__head">Set Shedule</h1>
    <div>
      <div class="d-flex calendar__header">
        <div class="calendar__legend calendar__legend_check-day">
          All day
        </div>
        <ul class="calendar__legend calendar__legend_times p-0">
          <li
            class="calendar__legend_time"
            v-for="time in times"
            :key="'legend_' + time"
          >
            {{ time }}
          </li>
        </ul>
      </div>
      <ul
        class="p-0"
        ref="calendar"
      >
        <li
          class="d-flex"
          v-for="(value, dayName, dayIndex) in days"
          :key="'day_' + dayIndex"
        >
          <div
            class="calendar__legend calendar__legend_day"
            :class="{ 'calendar_active': value.length }"
          >
            {{ dayName }}
          </div>
          <button
            class="calendar__check-day"
            :class="{ 'calendar_active': value.length && value[0].bt === 0 && value[0].et === 1439 }"
            @click="checkDay(dayName)"
          />
          <ul class="d-flex p-0">
            <li
              v-for="(time, timeIndex) in times"
              :key="'day_' + dayIndex + '_time_' + timeIndex"
            >
              <ul class="d-flex p-0">
                <li
                  v-for="hour in 3"
                  :key="hour"
                  class="calendar__day"
                  :class="{ 'calendar_active': isSelected(dayName, time, hour) }"
                  @click="checkHour(dayName, time, hour)"
                  @mouseenter="hoverMouse(dayName, time, hour)"
                />
              </ul>
            </li>
          </ul>
        </li>
      </ul>
    </div>
    <div class="calendar__actions">
      <button
        class="calendar__button"
        :disabled="isCanBeClear"
        @click="clear"
      >
        Clear
      </button>
      <button
        class="calendar__button"
        :disabled="!isCanBeSave"
        @click="saveSchedule"
      >
        Save changes
      </button>
    </div>
  </div>
</template>

<script>
  export default {
    name: 'CalendarSchedule',
    props: {
      checkedDay: {
        type: Object,
        default: () => ({})
      }
    },
    data () {
      return {
        times: ['00:00', '03:00', '06:00', '09:00', '12:00', '15:00', '18:00', '21:00'],
        days: {},
        emptyDays: { 'mo':[], 'tu':[], 'we':[], 'th':[], 'fr':[], 'sa':[], 'su':[] },
        isMouseClamped: false
      }
    },
    computed: {
      isCanBeClear () {
        return Object.keys(this.days).every(day => !this.days[day].length)
      },
      isCanBeSave () {
        return JSON.stringify(this.days) !== JSON.stringify(this.checkedDay)
      }
    },
    mounted () {
      window.addEventListener("mousedown", this.onMouseDown)
      window.addEventListener("mouseup", this.onMouseUp)
      if (Object.keys(this.checkedDay).length)
        this.days = JSON.parse(JSON.stringify(this.checkedDay))
      else this.days = JSON.parse(JSON.stringify(this.emptyDays))
    },
    beforeDestroy () {
      window.removeEventListener("mousedown", this.onMouseDown)
      window.removeEventListener("mouseup", this.onMouseUp)
    },
    methods: {
      isSelected (day, time, hour) {
        const currentHour = this.getBouneries(time, hour)
        return this.days[day].find(interval => interval.bt <= currentHour.bt && interval.et >= currentHour.et)
      },
      checkDay (day) {
        if (this.days[day] && this.days[day][0] && this.days[day][0].bt === 0 && this.days[day][0].et === 1439)
          this.days[day] = []
        else this.days[day] = [{ "bt": 0, "et": 1439 }]
      },
      getBouneries (time, hour) {
        const result = { bt: 0, et: 0 }
        const realTime = time.split(':')[0]
        result.bt = (+realTime + --hour) * 60
        result.et = result.bt + 59
        return result
      },
      checkHour (day, time, hour) {
        const checkedHour = this.getBouneries(time, hour)
        if (this.days[day].length) this.setHour(day, checkedHour)
        else this.days[day].push(checkedHour)
      },
      setHour (day, hour, isMouseEvents = false) {
        const isHourAlreadeChecked = this.days[day].find(interval => interval.bt <= hour.bt && interval.et >= hour.et)
        if (!!isHourAlreadeChecked && !isMouseEvents) this.removeHourFromDay(day, hour)
        else if (!isHourAlreadeChecked) this.addHourToDay(day, hour)
      },
      removeHourFromDay (day, hour) {
        const decombineDay = []
        this.days[day].forEach(interval => {
          const numHours = (interval.et + 1 - interval.bt) / 60
          for (let i = 0; i < numHours; i++)
            decombineDay.push([interval.bt  + (i * 60), interval.bt  + (i * 60) + 59])
        })
        const indexRemovedHour = decombineDay.findIndex(interval => interval[0] === hour.bt && interval[1] === hour.et)
        decombineDay.splice(indexRemovedHour, 1)
        const result = []
        decombineDay.forEach(interval => result.push({ "bt": interval[0], "et": interval[1] }))
        this.days[day] = result
      },
      addHourToDay (day, hour) {
        const combineDay = []
        this.days[day].forEach(time => combineDay.push(...Object.values(time)))
        combineDay.push(...Object.values(hour))
        combineDay.sort((a, b) => a - b)

        let i = 1
        while (i <= combineDay.length) {
          if (combineDay[i] === combineDay[i + 1] - 1) combineDay.splice(i, 2)
          else i += 2
        }

        const result = []
        for (let i = 0; i < combineDay.length; i += 2)
          result.push({ "bt": combineDay[i], "et": combineDay[i + 1] })
        this.days[day] = result
      },
      hoverMouse (day, time, hour) {
        if (!this.isMouseClamped) {
          return false
        }
        const hoveredHour = this.getBouneries(time, hour)
        this.setHour(day, hoveredHour, true)
      },
      onMouseDown () {
        this.isMouseClamped = true
      },
      onMouseUp () {
        this.isMouseClamped = false
      },
      clear () {
        this.days = JSON.parse(JSON.stringify(this.emptyDays))
      },
      saveSchedule () {
        this.$emit('save-schedule', this.days)
      }
    }
  }
</script>

<style lang="sass">
  html, body, div, span, applet, object, iframe,
  h1, h2, h3, h4, h5, h6, p, blockquote, pre,
  a, abbr, acronym, address, big, cite, code,
  del, dfn, em, img, ins, kbd, q, s, samp,
  small, strike, strong, sub, sup, tt, var,
  b, u, i, center,
  dl, dt, dd, ol, ul, li,
  fieldset, form, label, legend,
  table, caption, tbody, tfoot, thead, tr, th, td,
  article, aside, canvas, details, embed,
  figure, figcaption, footer, header, hgroup,
  menu, nav, output, ruby, section, summary,
  time, mark, audio, video
    margin: 0
    padding: 0
    border: 0
    font-size: 100%
    font: inherit
    vertical-align: baseline

  /* HTML5 display-role reset for older browsers */
  article, aside, details, figcaption, figure,
  footer, header, hgroup, menu, nav, section
    display: block

  body
    line-height: 1

  ol, ul
    list-style: none

  blockquote, q
    quotes: none

  blockquote:before, blockquote:after,
  q:before, q:after
    content: ''
    content: none

  table
    border-collapse: collapse
    border-spacing: 0

  $with-primitive: 25px
  $height-prmitive: 50px
  .d-flex
    display: flex
  .flex-center
    align-items: center
    justify-content: center
  .p-0
    padding: 0
  .calendar
    &_active
      background-color: lightgrey
      &.calendar__check-day
        position: relative
        &::after
          content: url("data:image/svg+xml; utf8, <svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 24 24'><path d='M12 2C6.5 2 2 6.5 2 12S6.5 22 12 22 22 17.5 22 12 17.5 2 12 2M12 20C7.59 20 4 16.41 4 12S7.59 4 12 4 20 7.59 20 12 16.41 20 12 20M16.59 7.58L10 14.17L7.41 11.59L6 13L10 17L18 9L16.59 7.58Z' fill='white' /></svg>")
          width: 30px
          height: 30px
          position: absolute
          margin: auto
          top: 0
          bottom: 0
          right: 0
          left: 0
    &__head
      text-align: left
      font-weight: bold
      font-size: 20px
      margin-top: 250px
      margin-bottom: 15px
    &__legend
      display: flex
      width: $with-primitive * 2
      font-size: 14px
      &_day
        height: $height-prmitive
        align-items: center
        justify-content: center
        text-transform: uppercase
        border: 1px solid lightgrey
      &_times
        width: auto
        justify-content: end
      &_time
        width: $with-primitive * 3
        text-align: left
        position: relative
        display: flex
        align-items: center
        &:after
          content: ''
          position: absolute
          left: -1px
          bottom: 0
          width: 2px
          height: 10px
          background-color: lightgrey
      &_check-day
        text-transform: uppercase
        margin-left: $with-primitive * 2
        width: $with-primitive * 2
        box-sizing: content-box
        text-align: center
    &__day
      width: $with-primitive
      height: $height-prmitive
      border: 1px solid lightgrey
    &__check-day
      width: $with-primitive * 2
      height: $height-prmitive
      background-color: grey
      border: 1px solid lightgrey
    &__actions
      margin-top: 20px
      display: flex
      justify-content: end
    &__button
      min-width: 100px
      padding: 10px
      background-color: lightgrey
      margin-left: 20px

</style>
