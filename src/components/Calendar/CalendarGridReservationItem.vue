<template>
  <div
    class="calendar-grid-reservation"
    :class="[
      dragging
        ? 'calendar-grid-reservation-dragging calendar-grid-reservation-edit'
        : 'calendar-grid-reservation-stopped',
      resizing ? 'calendar-grid-reservation-resizing' : '',
      reservation.clone ? 'calendar-grid-reservation-clone' : '',
    ]"
    :style="{
      top: computedPositionY,
      left: computedPositionX,
      width: minWidthAnimation ? minWidthAnimation : computedWidth + 'px',
      height: computedHeight,
    }"
  >
    <div
      class="calendar-grid-reservation-left calendar-grid-reservation-resize"
      @mousedown="onMouseDownResizeLeft"
    ></div>
    <div
      class="calendar-grid-reservation-content"
      @mousedown="onMouseDownWillMove"
      @mouseenter="handleMouseEnter"
      @mouseleave="handleMouseLeave"
    >
      <div class="calendar-grid-reservation-content-room-time">
        {{ dateStartFormatted + " - " + dateEndFormatted }}
      </div>
      <div class="calendar-grid-reservation-content-room-name">
        {{
          reservation.meeting_title
            ? reservation.meeting_title
            : "Reserva en " + reservation.room.name
        }}
      </div>

      <div class="calendar-grid-reservation-content-room-area">
        {{ computedBusiness }}
      </div>
      <div class="calendar-grid-reservation-content-room-customer">
        {{ computedCustomer }}
      </div>
    </div>
    <div
      class="calendar-grid-reservation-right calendar-grid-reservation-resize"
      @mousedown="onMouseDownExpandRight"
    ></div>
  </div>
</template>

<script lang="ts">
import { Component, Prop, Vue } from "vue-property-decorator";
import { DateTime } from "ts-luxon";

@Component
export default class CalendarGridReservation extends Vue {
  @Prop({
    default: () => {
      return {};
    },
  })
  private reservation!: any;
  @Prop({
    default: () => {
      return { hour: 0, minute: 0 };
    },
  })
  dateStart!: any;

  reservationStart = DateTime.fromMillis(this.reservation.starts_at).toFormat(
    "TT"
  );
  reservationEnd = DateTime.fromMillis(this.reservation.ends_at).toFormat("TT");

  @Prop({ default: 0 }) indexReservation!: number;
  @Prop({ default: 70 }) minHeight!: number;
  @Prop({ default: 100 }) widthCell!: number;
  @Prop({ default: [] }) private rooms!: Array<any>;

  @Prop({
    default: () => {
      return DateTime.now();
    },
  })
  dateSelected!: DateTime;

  dragging: boolean = false;
  resizing: boolean = false;
  offsetX: number = 0;
  offsetY: number = 0;
  tempX: number = 0;
  tempY: number = 0;
  minWidthAnimation: any = null;

  get dateStartFormatted(): string {
    return DateTime.fromMillis(this.reservation.starts_at)
      .setLocale("en")
      .setZone("America/Lima")
      .toFormat("t");
  }

  get dateEndFormatted(): string {
    return DateTime.fromMillis(this.reservation.ends_at)
      .setLocale("en")
      .setZone("America/Lima")
      .toFormat("t");
  }

  onMouseDownWillMove(event: MouseEvent): any {
    this.handleMouseLeave(event);
    const node = event.target as HTMLElement;
    this.offsetX = event.clientX - node.getBoundingClientRect().left;
    this.offsetY =
      event.clientY - node.getBoundingClientRect().top - event.offsetY;
    this.dragging = true;

    this.reservation.offsetX = this.offsetX;
    this.reservation.offsetY = this.offsetY;
    this.reservation.width = this.computedWidth + "px";
    this.reservation.indexReservation = this.indexReservation;

    this.$emit("handleWillMoveReservation", this.reservation);
  }

  onMouseDownResizeLeft(event: MouseEvent): void {
    this.handleMouseLeave(event);
    const node = event.target as HTMLElement;
    this.offsetX = event.clientX - node.getBoundingClientRect().left;
    this.offsetY =
      event.clientY - node.getBoundingClientRect().top - event.offsetY;
    this.resizing = true;

    this.reservation.offsetX = this.offsetX;
    this.reservation.offsetY = this.offsetY;
    this.reservation.width = null;
    this.reservation.indexReservation = this.indexReservation;

    this.$emit("handleWillResizeReservation", this.reservation);
  }

  onMouseDownExpandRight(event: MouseEvent): void {
    this.handleMouseLeave(event);
    const node = event.target as HTMLElement;
    this.offsetX = event.clientX - node.getBoundingClientRect().left;
    this.offsetY =
      event.clientY - node.getBoundingClientRect().top - event.offsetY;
    this.resizing = true;

    this.reservation.offsetX = this.offsetX;
    this.reservation.offsetY = this.offsetY;
    this.reservation.width = null;
    this.reservation.indexReservation = this.indexReservation;

    this.$emit("handleWillResizeReservation", this.reservation, "toRight");
  }

  // eslint-disable-next-line @typescript-eslint/no-unused-vars
  handleMouseEnter(event: MouseEvent): void {
    if (!this.reservation.clone && this.computedWidth < this.minHeight * 2) {
      this.minWidthAnimation = `${this.minHeight * 2}px`;
    }
  }

  // eslint-disable-next-line @typescript-eslint/no-unused-vars
  handleMouseLeave(event: MouseEvent): void {
    this.minWidthAnimation = null;
  }

  // eslint-disable-next-line @typescript-eslint/no-empty-function,@typescript-eslint/no-unused-vars
  onMouseUp(event: MouseEvent): void {
    this.dragging = false;
    this.resizing = false;
  }

  stopDrag(): any {
    this.$emit("handleWillStopMoveReservation");
    this.dragging = false;
    this.resizing = false;
  }

  get computedPositionY(): string {
    if (this.reservation?.top) {
      return `${this.reservation.top}px`;
    }

    if (this.reservation?.starts_at && this.reservation?.ends_at) {
      return `${this.getPositionYFromRoomID(this.reservation.room.id)}px`;
    }

    return `${0}px`;
  }

  get computedPositionX(): string {
    if (this.reservation?.left) {
      return `${this.reservation.left}px`;
    }

    if (this.reservation?.starts_at && this.reservation?.ends_at) {
      return `${this.getPositionXFromDate(this.reservation?.starts_at)}px`;
    }

    return `${0}px`;
  }

  get computedWidth(): number {
    return (
      this.getPositionXFromDate(this.reservation.ends_at) -
      this.getPositionXFromDate(this.reservation.starts_at)
    );
  }

  get computedHeight(): string {
    return `${this.minHeight - 5}px`;
  }

  get computedCustomer(): string {
    return `${this.reservation?.customer?.name || ""} ${
      this.reservation?.customer?.last_name || ""
    }`;
  }

  get computedBusiness(): string {
    return `${this.reservation?.room?.business?.name || ""} ${
      this.reservation?.room?.business?.name || ""
    }`;
  }

  getPositionYFromRoomID(roomID: number): number {
    let posY = 0;
    for (const index in this.rooms) {
      if (this.rooms[+index].id === roomID) {
        posY = +index * this.minHeight + 2;
      }
    }
    return posY;
  }

  getPositionXFromDate(timestamp: number): number {
    const luxonDate = DateTime.fromMillis(timestamp);
    let posX = 0;

    let luxonHour = luxonDate.hour;
    const luxonMinute = luxonDate.minute;

    if (luxonDate.day > this.dateSelected.day) {
      luxonHour = 24;
    }

    const hoursWidth = (luxonHour - this.dateStart.hour) * this.widthCell;
    const minutesWidth = (luxonMinute * this.widthCell) / 60;

    posX = hoursWidth + minutesWidth;

    return posX;
  }

  mounted(): any {
    window.addEventListener("mouseup", this.stopDrag);
  }
}
</script>
