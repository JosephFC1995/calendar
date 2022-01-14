<template>
  <div class="calendar-grid-header" id="calendar-grid-header">
    <div class="calendar-grid-header-hours">
      <div class="calendar-grid-header-hour hour hour-first"></div>
      <div
        class="calendar-grid-header-hour calendar-grid-header-hour-block hour"
        v-for="(hour, index) in HoursRanges"
        :key="index"
        :data-time-start="hour.time"
        :data-time-end="
          HoursRanges[index + 1] ? HoursRanges[index + 1].time : ''
        "
        :data-hour="hour.hour"
        :style="{ flex: '0 0 ' + widthCell + 'px' }"
      ></div>
    </div>
  </div>
</template>

<script lang="ts">
import { Component, Prop, Vue } from "vue-property-decorator";
import { DateTime, Duration } from "ts-luxon";

@Component
export default class CalendarGridHeader extends Vue {
  @Prop({
    default: () => {
      return { hour: 0, minute: 0 };
    },
  })
  dateStart!: any;
  @Prop({ default: 100 }) widthCell!: number;

  dateNow = DateTime.now().setLocale("es").setZone("America/Lima");

  get HoursRanges(): any {
    let times: any = [];
    const startHour = this.dateNow.set({
      hours: this.dateStart.hour,
      minutes: this.dateStart.minute,
      seconds: 0,
    });
    let hour = Duration.fromObject({
      hours: this.dateStart.hour,
      minutes: this.dateStart.minute,
    }).as("minutes");

    let maxHours = Duration.fromObject({
      hours: 23 - this.dateStart.hour,
      minutes: 60 - this.dateStart.minute,
    }).as("minutes");

    for (let i = 0; hour < maxHours; i++) {
      const dur = Duration.fromObject({ hours: i });
      hour = dur.as("minutes");
      const temp = startHour.plus(dur);
      times.push({
        time: temp.toFormat("dd-LL-y T"),
        hour: temp.toFormat("T"),
      });
    }
    times = times.slice(0, times.length - 1);
    return times;
  }
}
</script>

<style lang="scss" scoped>
.calendar {
  &-grid {
    &-header {
      margin-top: 0;
      position: sticky;
      top: 0;
      z-index: 3;

      &-hours {
        display: flex;
      }

      &-hour {
        &:first-child {
          border-right: 2px solid #f3f7f9;
          border-top: 2px solid #f3f7f9;
          border-bottom: 2px solid #f3f7f9;
          flex: 0 0 200px;
          position: sticky;
          left: 0;
          background: white;
          z-index: 2;
        }

        min-height: 50px;
        flex: 0 0 100px;
        border-right: 1px dashed #bcbcbc;
        border-top: 2px solid #f3f7f9;
        border-bottom: 2px solid #f3f7f9;
        position: relative;
        box-sizing: border-box;
        user-select: none;
        background-color: white;

        &::before {
          content: attr(data-hour);
          font-size: 12px;
          line-height: 14px;
          color: #223345;
          top: 50%;
          transform: translateY(-50%);
          position: absolute;
          left: 5px;
          font-weight: 600;
        }
      }
    }
  }
}
</style>
