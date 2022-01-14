<template>
  <div class="calendar-grid-hour-list">
    <div
      class="calendar-grid-content-hours"
      :data-room-id="'hour-room:' + room.id"
      :id="'hour-room-' + room.id"
    >
      <div
        class="calendar-grid-content-list-hour hour hour-room"
        :class="{ noIsHour: hour.noIsHour }"
        v-for="(hour, index) in HoursRanges"
        :key="index"
        :data-time-start="hour.time"
        :data-time-end="
          HoursRanges[index + 1] ? HoursRanges[index + 1].time : ''
        "
        :data-hour="hour.hour"
        :style="{
          height: minHeight + 'px',
          flex: '0 0 ' + widthCell / 2 + 'px',
        }"
      ></div>
    </div>
  </div>
</template>

<script lang="ts">
import { Component, Prop, Vue } from "vue-property-decorator";
import { DateTime, Duration } from "ts-luxon";
import CalendarGridReservation from "@/components/Calendar/CalendarGridReservationItem.vue";
@Component({
  components: { CalendarGridReservation },
})
export default class CalendarGridRoom extends Vue {
  @Prop({
    default: () => {
      return { hour: 0, minute: 0 };
    },
  })
  dateStart!: any;
  @Prop({ default: {} }) room!: any;
  @Prop({ default: 70 }) minHeight!: number;
  @Prop({ default: 100 }) widthCell!: number;
  @Prop({
    default: () => {
      return DateTime.now();
    },
  })
  dateSelected!: DateTime;

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

    let minute = 0;
    for (let i = 0; hour < maxHours; i += 30) {
      const dur = Duration.fromObject({ minutes: i });
      if (minute === 0) minute = 30;
      hour = dur.as("minutes");
      const temp = startHour.plus({
        minutes: hour,
      });
      times.push({
        time: temp.toFormat("dd-LL-y TT"),
        hour: temp.toFormat("TT"),
        noIsHour: i % 60 != 0,
      });
    }
    times = times.slice(0, times.length - 1);
    return times;
  }
}
</script>
