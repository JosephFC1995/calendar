<template>
  <div
    class="calendar-grid-time"
    :time="currentTime"
    :style="{ left: leftSize + 'px' }"
    :class="[computedDisabledInteractions ? 'disabled-interaction' : '']"
    @mouseenter="handleMouseEnter"
    @mouseleave="handleMouseLeave"
    @mousemove="handleMouseMove"
  >
    <div class="calendar-grid-time-circle"></div>
    <div class="calendar-grid-time-line"></div>
    <div
      class="calendar-grid-time-popup"
      v-if="showTimePopup"
      :style="{ top: topPopUpPosition + 'px' }"
    >
      {{ currentTime }}
    </div>
  </div>
</template>

<script lang="ts">
import { Component, Prop, Vue } from "vue-property-decorator";
import { DateTime, Duration } from "ts-luxon";

@Component
export default class CalendarGridTime extends Vue {
  @Prop({
    default: () => {
      return { hour: 0, minute: 0 };
    },
  })
  dateStart!: any;
  @Prop({ default: 100 }) widthCell!: number;
  @Prop({ default: false }) disabledInteraction!: boolean;

  leftSize: number = 197;
  topPopUpPosition: number = 0;
  heightPopUp: number = 0;
  currentTime: any = DateTime.now()
    .setLocale("es")
    .setZone("America/Lima")
    .toFormat("TT");
  intervalClock: any;
  durationNow: number = 0;
  durationTwentyForHours = Duration.fromObject({ hours: 24 }).as("minutes");
  widthHeaderCalendar: number = 0;
  showTimePopup: boolean = false;

  leftCurrentTime(durationNow: number): any {
    const tempWidth =
      (durationNow * this.widthHeaderCalendar) / this.durationTwentyForHours;
    this.leftSize =
      Math.round(tempWidth * 100) / 100 +
      195 -
      this.dateStart.hour * this.widthCell;
  }

  startClock(): any {
    this.intervalClock = setInterval(() => {
      this.currentTime = DateTime.now()
        .setLocale("es")
        .setZone("America/Lima")
        .toFormat("TT");
      const { hour, minute } = DateTime.now()
        .setLocale("es")
        .setZone("America/Lima")
        .toObject();
      const dur = Duration.fromObject({
        hours: hour,
        minutes: minute,
      });
      this.durationNow = dur.as("minutes");
      this.leftCurrentTime(this.durationNow);
    }, 1000);
  }

  get computedDisabledInteractions(): boolean {
    return this.disabledInteraction;
  }

  stopClock(): any {
    clearInterval(this.intervalClock);
  }

  handleMouseEnter(): any {
    this.showTimePopup = true;
  }

  handleMouseLeave(): any {
    this.showTimePopup = false;
  }

  handleMouseMove($event: MouseEvent): any {
    if ($event.offsetY > 0) {
      this.topPopUpPosition = $event.offsetY;
    }
  }

  mounted(): any {
    this.startClock();
    // const headerElement: any = document.getElementsByClassName(
    //   "calendar-grid-header-hour-block"
    // );
    // let tempWidth = 0;
    // for (let i = 0; i < headerElement.length - 1; i++) {
    //   const element = headerElement[i];
    //   tempWidth += element.offsetWidth;
    // }
    const heightContent = document.getElementsByClassName(
      "calendar-grid-content"
    );
    this.heightPopUp =
      heightContent.length > 0 ? heightContent[0].scrollHeight - 55 : 0;
    this.widthHeaderCalendar = this.widthCell * 24;
  }

  beforeDestroy(): any {
    this.stopClock();
  }
}
</script>

<style lang="scss" scoped>
.calendar {
  &-grid {
    &-time {
      height: 100%;
      position: absolute;
      top: 48px;
      transition: 0.3s left;
      z-index: 1;

      &.disabled-interaction {
        pointer-events: none;
      }

      &-circle {
        height: 10px;
        width: 10px;
        background: #00a8cd;
        border-radius: 50%;
        z-index: 5;
        position: sticky;
        top: 50px;
      }

      &-line {
        width: 1px;
        background: #00a8cd;
        height: 100%;
        position: absolute;
        left: 50%;
        transform: translateX(-50%);
        top: 0;
      }

      &-popup {
        position: absolute;
        top: 0;
        right: 20px;
        font-size: 12px;
        font-weight: 600;
        color: white !important;
        background: #00a8cd;
        font-variant: tabular-nums;
        padding: 2px 5px;
        border-radius: 5px;
        pointer-events: none;
      }
    }
  }
}
</style>
