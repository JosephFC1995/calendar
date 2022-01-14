<template>
  <div
    class="calendar-grid-reservation"
    :class="[
      dragging
        ? 'calendar-grid-reservation-dragging'
        : 'calendar-grid-reservation-stopped',
    ]"
    :style="{ left: leftPosition + 'px', width: widthPosition + 'px' }"
    @mousedown="startDrag"
    @mouseup="stopDrag"
    @mouseleave="stopDrag"
    @mousemove="doDrag"
  ></div>
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

  dragging: boolean = false;
  leftBackupPosition: number = 0;
  leftPosition: number = 0;
  rightPosition: number = 0;
  widthPosition: number = 5;

  movePixels: number = 0;

  startDrag(): any {
    this.dragging = true;
  }

  stopDrag(): any {
    this.dragging = false;
    this.movePixels = 0;
    this.leftBackupPosition = this.leftPosition;
  }

  doDrag(event: any): any {
    if (this.dragging) {
      this.movePixels += event.movementX;
      // eslint-disable-next-line no-empty
      if (this.movePixels % 55 == 0) {
      }
      this.leftPosition = this.leftBackupPosition + this.movePixels;

      console.log(this.movePixels);
    }
  }

  mounted(): any {
    window.addEventListener("mouseup", this.stopDrag);
    const panel: any = document.getElementById(
      `room-${this.reservation.room.id}`
    );
    const staticStart = DateTime.fromMillis(
      this.reservation.starts_at
    ).toFormat("TT");
    const staticEnd = DateTime.fromMillis(this.reservation.ends_at).toFormat(
      "TT"
    );
    const gridStart = panel.querySelectorAll(
      "[data-hour='" + staticStart + "']"
    );
    const gridEnd = panel.querySelectorAll("[data-hour='" + staticEnd + "']");
    this.leftPosition = gridStart[0].offsetLeft - 200;
    this.leftBackupPosition = gridStart[0].offsetLeft - 200;
    this.rightPosition = gridEnd[0].offsetLeft - 200 - 5;

    this.widthPosition = this.rightPosition - this.leftPosition;
    console.log(this.widthPosition);
    console.log(this.leftPosition, this.rightPosition);
    console.log("=====");
  }
}
</script>
