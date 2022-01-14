<template>
  <div
    class="calendar-grid-reservations-grid"
    ref="roomBookings"
    @mousemove="onMouseMoveFunc"
    @mousedown.self="onMouseDownFunc"
    @mouseup="onMouseUpFunc"
  >
    <CalendarGridReservationItem
      v-for="(reservation, index) in reservations"
      :reservation="reservation"
      :key="index"
      :indexReservation="index"
      :minHeight="minHeight"
      :widthCell="widthCell"
      :rooms="rooms"
      :dateStart="dateStart"
      :dateSelected="dateSelected"
      @handleWillMoveReservation="handleWillMoveReservation"
      @handleWillStopMoveReservation="handleWillStopMoveReservation"
      @handleWillResizeReservation="handleWillResizeReservation"
    />

    <CalendarGridReservationItem
      v-if="reservationTarget"
      :reservation="reservationTarget"
      :minHeight="minHeight"
      :widthCell="widthCell"
      :rooms="rooms"
      :dateStart="dateStart"
      :dateSelected="dateSelected"
    />

    <CalendarGridReservationItem
      v-if="reservationCreate"
      :reservation="reservationCreate"
      :minHeight="minHeight"
      :widthCell="widthCell"
      :rooms="rooms"
      :dateStart="dateStart"
      :dateSelected="dateSelected"
    />

    <CalendarGridReservationItem
      v-if="reservationCreateTarget"
      :reservation="reservationCreateTarget"
      :minHeight="minHeight"
      :widthCell="widthCell"
      :rooms="rooms"
      :dateStart="dateStart"
      :dateSelected="dateSelected"
    />
  </div>
</template>

<script lang="ts">
import { Component, Prop, PropSync, Ref, Vue } from "vue-property-decorator";
import CalendarGridReservationItem from "@/components/Calendar/CalendarGridReservationItem.vue";
import { DateTime } from "ts-luxon";

@Component({
  components: { CalendarGridReservationItem },
})
export default class CalendarGridReservation extends Vue {
  @Prop({
    default: () => {
      return { hour: 0, minute: 0 };
    },
  })
  dateStart!: any;
  @Prop({ type: Number, default: 100 }) widthCell!: number;
  @Prop({ default: [] }) private reservations!: Array<any>;
  @Prop({ default: [] }) private rooms!: Array<any>;
  @Prop({ default: 70 }) minHeight!: number;
  @Prop({
    default: () => {
      return DateTime.now();
    },
  })
  dateSelected!: DateTime;
  @PropSync("reservationNew") reservationCreate!: any;
  @Ref("roomBookings") readonly roomBookings!: HTMLButtonElement;

  draggingReservation: boolean = false;
  resizeReservation: boolean = false;
  createReservation: boolean = false;
  reservationTarget: any = null;
  reservationCreateTarget: any = null;

  movingTarget = { x: 0, y: 0 };
  fromExpanding: string = "";

  onMouseMoveFunc(event: MouseEvent): any {
    if (
      !this.draggingReservation &&
      !this.resizeReservation &&
      !this.createReservation
    )
      return;
    const pos = this.getRelativePosition(event);
    if (!pos) {
      return;
    }
    this.onDragAt(pos);
  }

  onMouseDownFunc(event: MouseEvent): any {
    if (!this.draggingReservation && !this.resizeReservation) {
      this.createReservation = true;
    }
    const pos = this.getRelativePosition(event);
    if (!pos) {
      return;
    }
    this.onDragStartedAt(pos);
  }

  onMouseUpFunc($event: MouseEvent): any {
    if (this.draggingReservation || this.resizeReservation) {
      const originalReservation =
        this.reservations[this.reservationTarget.indexReservation];

      this.reservations[this.reservationTarget.indexReservation] = {
        ...this.reservationTarget,
        clone: false,
        resize: false,
        move: false,
      };

      if (
        originalReservation.starts_at === this.reservationTarget.starts_at &&
        originalReservation.ends_at === this.reservationTarget.ends_at &&
        originalReservation.room.id === this.reservationTarget.room.id
      ) {
        this.reservationTarget = null;
        return;
      }

      if (this.resizeReservation) {
        this.$emit("resizeBooking", this.reservationTarget);
        this.reservationTarget = null;
        return;
      }
      this.$emit("moveBooking", this.reservationTarget);
      this.reservationTarget = null;
      return;
    }

    if (this.createReservation) {
      this.reservationCreate = this.reservationCreateTarget;
      this.reservationTarget = null;
      this.reservationCreateTarget = null;
      return;
    }
  }

  // eslint-disable-next-line @typescript-eslint/explicit-module-boundary-types
  onDragAt({ x, y }: any): void {
    if (this.draggingReservation || this.resizeReservation) {
      if (this.resizeReservation) {
        const newStartDate: DateTime = this.getDateByPx(x);
        const newEndDate: DateTime = this.getDateByPx(x, true);

        const reservationTargetBackup = JSON.parse(
          JSON.stringify(this.reservationTarget)
        );

        if (this.fromExpanding === "toRight") {
          reservationTargetBackup.ends_at = newEndDate.toMillis();
          const nb = newStartDate.minus({ minute: 15 });

          if (nb.toMillis() < this.reservationTarget.starts_at) {
            return;
            // TODO permit transform start_at to end_at
            // reservationTargetBackup.starts_at = newStartDate.toMillis();
            // const minutesPerSection = 60 / 4;
            // reservationTargetBackup.ends_at = DateTime.fromMillis(
            //   this.reservationTarget.starts_at
            // )
            //   .plus({ minute: minutesPerSection })
            //   .toMillis();
          }

          if (
            DateTime.fromMillis(
              reservationTargetBackup.starts_at
            ).toMillis() ===
            DateTime.fromMillis(reservationTargetBackup.ends_at).toMillis()
          ) {
            const minutesPerSection = 60 / 4;
            reservationTargetBackup.ends_at = DateTime.fromMillis(
              reservationTargetBackup.starts_at
            )
              .plus({ minute: minutesPerSection })
              .toMillis();
          }
        } else {
          reservationTargetBackup.starts_at = newStartDate.toMillis();
          const na = newStartDate.plus({ minute: 15 });

          if (na.toMillis() > this.reservationTarget.ends_at) {
            return;
            // TODO permit transform end_at to start_at
            // reservationTargetBackup.ends_at = newEndDate.toMillis();
            // const minutesPerSection = 60 / 2;
            // reservationTargetBackup.starts_at = DateTime.fromMillis(
            //   this.reservationTarget.ends_at
            // )
            //   .minus({ minute: minutesPerSection })
            //   .toMillis();
          }
        }

        this.limitNewBookingIfNeeded(
          reservationTargetBackup,
          DateTime.fromMillis(this.reservationTarget.starts_at).toMillis(),
          DateTime.fromMillis(this.reservationTarget.ends_at).toMillis(),
          this.getRoomByPy(this.movingTarget.y)
        );

        reservationTargetBackup.clone = true;
        reservationTargetBackup.move = false;
        reservationTargetBackup.resize = true;

        this.reservationTarget = reservationTargetBackup;

        return;
      }

      const movePixelX = x - this.reservationTarget.offsetX;
      const movePixelY = y - this.reservationTarget.offsetY;

      const movingFloatingBooking = Object.assign({}, this.reservationTarget);
      movingFloatingBooking.id = "movingFloatingBookingId";
      movingFloatingBooking.top = movePixelY;
      movingFloatingBooking.left = movePixelX;

      const newStartDate: DateTime = this.getDateByPx(movePixelX);
      const diffDates =
        this.reservationTarget.ends_at - this.reservationTarget.starts_at;

      const newBooking = JSON.parse(JSON.stringify(this.reservationTarget));
      newBooking.id = "movingBooking";
      newBooking.starts_at = newStartDate.toMillis();
      newBooking.ends_at = newStartDate.toMillis() + diffDates;
      newBooking.room.id = this.getRoomByPy(movePixelY);

      if (this.isMovingBookingOverlapping(newBooking)) {
        return;
      }

      const newEndDate: number = newStartDate.toMillis() + diffDates;

      const limitStartDate = this.dateSelected.set({
        hour: this.dateStart.hour,
        minute: this.dateStart.minute,
      });

      const limitEndDate = this.dateSelected.set({
        hour: 24,
        minute: 0,
      });

      this.reservationTarget.room.id = this.getRoomByPy(movePixelY);
      if (
        newStartDate.toMillis() >= limitStartDate.toMillis() &&
        newEndDate <= limitEndDate.toMillis()
      ) {
        this.reservationTarget.starts_at = newStartDate.toMillis();
        this.reservationTarget.ends_at = newEndDate;
        this.reservationTarget.clone = true;
        this.reservationTarget.move = true;
        this.reservationTarget.resize = false;
      }
      return;
    }

    const startDate = this.getDateByPx(this.movingTarget.x);

    const newBooking = {
      id: null,
      // roomId: this.getRoomByPosY(posY),
      starts_at: startDate.toMillis(),
      ends_at: startDate.toMillis(),
      clone: true,
      move: false,
      resize: false,
      room: { id: this.getRoomByPy(this.movingTarget.y) },
    };

    const initialStartD = this.getDateByPx(this.movingTarget.x);
    const initialEndD = this.getDateByPx(this.movingTarget.x, true);

    const newStartDate = this.getDateByPx(x);
    const newEndDate = this.getDateByPx(x, true);

    const newBookingBack = newBooking;
    newBookingBack.starts_at = initialStartD.toMillis();
    newBookingBack.ends_at = newEndDate.toMillis();

    if (newStartDate < initialStartD) {
      newBookingBack.starts_at = newStartDate.toMillis();
      newBookingBack.ends_at = initialEndD.toMillis();
    }

    // eslint-disable-next-line no-empty
    if (newBookingBack.starts_at === newBookingBack.ends_at) {
    }

    this.limitNewBookingIfNeeded(
      newBookingBack,
      initialStartD.toMillis(),
      initialEndD.toMillis(),
      this.getRoomByPy(this.movingTarget.y)
    );

    this.reservationCreateTarget = newBookingBack;

    return;
  }

  limitNewBookingIfNeeded(
    newBooking: any,
    startDate: any,
    endDate: any,
    roomID: any
  ): void {
    let limitStart = this.startDayLimit;
    let limitEnd = this.endDayLimit;

    for (const reservation in this.reservations) {
      // eslint-disable-next-line @typescript-eslint/no-unused-vars
      if (this.reservations[reservation].room.id === roomID) {
        if (startDate >= this.reservations[reservation].ends_at) {
          const newStartLimit = this.reservations[reservation].ends_at;
          if (!limitStart) {
            limitStart = newStartLimit;
          }
          if (limitStart && newStartLimit > limitStart) {
            limitStart = newStartLimit;
          }
        }
        if (endDate <= this.reservations[reservation].starts_at) {
          const newEndLimit = this.reservations[reservation].starts_at;

          if (limitEnd === null || (limitEnd && newEndLimit < limitEnd)) {
            limitEnd = newEndLimit;
          }
        }
      }
    }

    if (limitStart && newBooking.starts_at < limitStart) {
      newBooking.starts_at = limitStart;
    }

    if (limitEnd && newBooking.ends_at > limitEnd) {
      newBooking.ends_at = limitEnd;
    }
  }

  isMovingBookingOverlapping(newMovingBooking: any): boolean {
    if (
      newMovingBooking.starts_at < this.startDayLimit ||
      newMovingBooking.ends_at > this.endDayLimit ||
      newMovingBooking.room.id == null
    ) {
      return true;
    }

    for (const reservation in this.reservations) {
      if (
        newMovingBooking.room.id !== this.reservations[reservation].room.id ||
        this.reservationTarget.id === this.reservations[reservation].id
      ) {
        continue;
      }

      if (
        newMovingBooking.starts_at >=
          this.reservations[reservation].starts_at &&
        newMovingBooking.starts_at < this.reservations[reservation].ends_at
      ) {
        return true;
      }
      if (
        newMovingBooking.ends_at > this.reservations[reservation].starts_at &&
        newMovingBooking.ends_at <= this.reservations[reservation].ends_at
      ) {
        return true;
      }
      if (
        newMovingBooking.starts_at < this.reservations[reservation].starts_at &&
        newMovingBooking.ends_at > this.reservations[reservation].ends_at
      ) {
        return true;
      }
    }

    return false;
  }

  get startDayLimit(): number {
    const a = this.dateSelected.set({
      hour: this.dateStart.hour,
      minute: 0,
      second: 0,
      millisecond: 0,
    });
    return a.toMillis();
  }

  get endDayLimit(): number {
    const endDayL = this.dateSelected.plus({ day: 1 });
    return endDayL.toMillis();
  }

  getRoomByPy(y: number): string {
    const roomIndex = Math.floor(y / this.minHeight);
    if (roomIndex in this.rooms) {
      return this.rooms[roomIndex].id;
    }
    return "";
  }

  getDateByPx(x: number, toCeiling: boolean = false): DateTime {
    const divHourA = x / this.widthCell;
    const hourStepA = toCeiling
      ? +(Math.ceil(divHourA * 4) / 4).toFixed(2) * 4
      : +(Math.floor(divHourA * 4) / 4).toFixed(2) * 4;

    const hourGroupCellsWithDecimals = hourStepA / 4;

    const hourGroupCells = Math.floor(hourGroupCellsWithDecimals);

    const hourSectionCells = hourStepA - hourGroupCells * 4;

    const nh = this.dateStart.hour + hourGroupCells;

    const nm = (60 / 4) * hourSectionCells;

    return DateTime.fromFormat(
      this.dateSelected.toFormat("dd-LL-y") + " " + nh + ":" + nm,
      "dd-LL-y h:m"
    );
  }

  // eslint-disable-next-line @typescript-eslint/explicit-module-boundary-types
  onDragStartedAt({ x, y }: any): void {
    this.movingTarget = { x, y };
  }

  getRelativePosition(event: MouseEvent): any {
    if (!this.roomBookings) {
      return null;
    }
    const rect = this.roomBookings.getBoundingClientRect();
    const x = event.clientX - rect.left;
    const y = event.clientY - rect.top;

    return { x, y };
  }

  // eslint-disable-next-line @typescript-eslint/explicit-module-boundary-types
  handleWillMoveReservation(reservation: any): void {
    this.draggingReservation = true;
    this.reservationTarget = {
      hidden: false,
      ...reservation,
      room: { ...reservation.room },
      top: 0,
      left: 0,
      clone: true,
      move: false,
      resize: false,
    };
    this.$emit("activeInteraction", true);
  }

  handleWillResizeReservation(reservation: any, toRight: string): void {
    this.resizeReservation = true;
    this.fromExpanding = toRight ? "toRight" : "toLeft";
    this.reservationTarget = {
      hidden: false,
      ...reservation,
      room: { ...reservation.room },
      top: 0,
      left: 0,
      clone: true,
      move: false,
      resize: false,
    };
    this.$emit("activeInteraction", true);
  }

  handleWillStopMoveReservation(): void {
    this.draggingReservation = false;
    this.resizeReservation = false;
    this.createReservation = false;
    this.$emit("activeInteraction", false);
    // this.reservationTarget = null;
  }
}
</script>

<style lang="scss" scoped>
.calendar {
  &-grid {
    &-reservations {
      &-grid {
        position: absolute;
        left: 0;
        top: 0;
        width: 100%;
        height: 100%;
        color: black;
      }
    }
  }
}
</style>
