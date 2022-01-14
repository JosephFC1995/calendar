<template>
  <div class="calendar-grid-rooms-lists calendar-grid-left">
    <div
      class="calendar-grid-content-rooms"
      v-for="(room, index) in rooms"
      :key="index"
      :data-room-id="room.id"
      :id="'room-list-' + room.id"
    >
      <div
        class="calendar-grid-content-list-room"
        :style="{ height: minHeight + 'px' }"
        :title="'Sala de billar'"
      >
        <div class="room-info room-detail">
          <div class="room-info-type">
            <img
              src="@/assets/ic_type_room_co.png"
              alt="Capacity"
              class="ico"
            />
            Sala de reunión
          </div>
          <div class="room-info-name">{{ room.name }}</div>
          <div class="room-info-capacity">
            <img src="@/assets/ic_capacity_co.png" alt="Capacity" class="ico" />
            {{ room.capacity }}
          </div>
        </div>
      </div>
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
  @Prop({ default: {} }) rooms!: any;
  @Prop({ default: 70 }) minHeight!: number;
  @Prop({ default: 100 }) widthCell!: number;
  @Prop({
    default: () => {
      return [];
    },
  })
  private reservations!: Array<any>;

  dateNow = DateTime.now().setLocale("es").setZone("America/Lima");
  widthCellReservations = (24 - this.dateStart.hour) * this.widthCell - 4;

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
      });
    }
    return times;
  }
}
</script>
