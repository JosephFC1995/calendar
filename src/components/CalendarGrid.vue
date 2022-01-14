<template>
  <div class="calendar-grid" :id="idComponent" :class="[idComponent]">
    <div class="calendar-grid-top">
      <div class="left"></div>
      <div class="center today-month-name">{{ dateFormatHeaderSelected }}</div>
      <div class="right actions">{{ dateStart }}</div>
    </div>
    <div class="calendar-grid-auxiliary">
      <div class="calendar-grid-content">
        <CalendarGridHeader :dateStart="dateStart" :widthCell="widthCell" />
        <div class="calendar-grid-block">
          <CalendarGridRoomList
            :dateStart="dateStart"
            :minHeight="minHeight"
            :rooms="rooms"
            :widthCell="widthCell"
          />
          <div class="calendar-grid-right">
            <CalendarGridRoom
              v-for="(room, index) in rooms"
              :key="index"
              :dateStart="dateStart"
              :minHeight="minHeight"
              :room="room"
              :widthCell="widthCell"
              :dateSelected="dateSelected"
            />
            <CalendarGridReservation
              :widthCell="widthCell"
              :dateStart="dateStart"
              :style="{ width: computedWidth }"
              :rooms="rooms"
              :reservations="reservations"
              :minHeight="minHeight"
              :dateSelected="dateSelected"
              @moveBooking="onEditingBooking"
              @resizeBooking="onResizeBooking"
              @activeInteraction="handleActiveInteraction"
              :reservationNew.sync="reservationCreate"
            />
          </div>
        </div>
        <CalendarGridTime
          :dateStart="dateStart"
          :widthCell="widthCell"
          :disabledInteraction="disabledInteraction"
        />
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { Component, Prop, PropSync, Vue } from "vue-property-decorator";
import { DateTime } from "ts-luxon";
import CalendarGridHeader from "@/components/Calendar/CalendarGridHeader.vue";
import CalendarGridRoom from "@/components/Calendar/CalendarGridRoomHour.vue";
import CalendarGridTime from "@/components/Calendar/CalendarGridTime.vue";
import CalendarGridRoomList from "@/components/Calendar/CalendarGridRoomList.vue";
import CalendarGridReservation from "@/components/Calendar/CalendarGridReservation.vue";

const randomString = () => (Math.random() + 1).toString(36).substring(2);

@Component({
  components: {
    CalendarGridReservation,
    CalendarGridRoomList,
    CalendarGridTime,
    CalendarGridRoom,
    CalendarGridHeader,
  },
})
export default class CalendarGrid extends Vue {
  @Prop({ default: "calendar-grid" }) private id!: string;
  @Prop({ default: [] }) private rooms!: Array<any>;
  @Prop({ default: [] }) private reservations!: any;
  @Prop({
    default: () => {
      return { hour: 0, minute: 0 };
    },
  })
  dateStart!: any;
  @Prop({ default: 70 }) minHeight!: number;
  @Prop({ default: 100 }) widthCell!: number;
  @PropSync("reservationNew") reservationCreate!: any;

  dateSelected = DateTime.local(2022, 1, 10, 8, 10, 23)
    .setLocale("es")
    .setZone("America/Lima");

  disabledInteraction: boolean = false;

  // dateSelected = DateTime.now().setLocale("es").setZone("America/Lima");

  get idComponent(): string {
    return this.id + "-" + randomString();
  }

  get computedWidth(): string {
    return (24 - this.dateStart.hour) * this.widthCell + "px";
  }

  get dateFormatHeaderSelected(): string {
    return this.dateSelected.toFormat("LLL dd, yyyy");
  }

  // eslint-disable-next-line @typescript-eslint/explicit-module-boundary-types
  onEditingBooking($event: any): void {
    this.$emit("reservationMove", $event);
  }

  // eslint-disable-next-line @typescript-eslint/explicit-module-boundary-types
  onResizeBooking($event: any): void {
    this.$emit("reservationResize", $event);
  }

  handleActiveInteraction($event: boolean): void {
    this.disabledInteraction = $event;
  }
}
</script>

<style scoped lang="scss">
* {
  margin: 0;
}

.calendar {
  &-grid {
    background: #fff;
    border-radius: 15px;

    &-block {
      display: flex;
    }

    &-left {
      flex: 0 0 200px;
      display: flex;
      flex-direction: column;
    }

    &-right {
      flex: 1;
      position: relative;
    }

    &-auxiliary {
      max-height: 80vh;
      overflow: scroll;
      color: #00000000;
      transition: color 0.3s;

      &:hover {
        color: #bcbcbc;
      }

      &::-webkit-scrollbar {
        width: 14px;
      }

      &::-webkit-scrollbar-thumb {
        background-clip: content-box;
        border: 4px solid transparent;
        border-radius: 7px;
        box-shadow: inset 0 0 0 10px;
      }

      &::-webkit-scrollbar-button {
        width: 0;
        height: 0;
        display: none;
      }

      &::-webkit-scrollbar-corner {
        background-color: transparent;
      }
    }

    * {
      font-family: Arial, serif;
    }

    &-content {
      max-width: 100%;
      position: relative;
      height: 100%;
    }

    &-top {
      display: flex;
      width: 98%;
      justify-content: space-between;
      min-height: 80px;
      align-items: center;
      margin: 0 auto;
      background-color: #fff;

      .today-month-name {
        font-weight: 600;
        font-size: 20px;
        line-height: 23px;
        text-align: justify;
        color: #00a8cd;
        text-transform: capitalize;
      }
    }

    ::v-deep {
      .calendar {
        &-grid {
          &-content {
            margin-top: 0;

            &-hours {
              display: flex;
              position: relative;
            }

            &-reservations {
              position: absolute;
              left: 202px;
              bottom: 2px;
              top: 2px;
              background: transparent;
            }

            &-list-hour {
              min-height: 70px;
              width: 100%;
              border-top: none;
              border-bottom: 1px dashed #bcbcbc;
              cursor: pointer;
              box-sizing: border-box;
              user-select: none;
              padding: 2px;

              &.noIsHour {
                border-right: 1px dashed #bcbcbc;
              }

              &:last-child {
                border-right: 1px dashed #bcbcbc;
              }
            }

            &-list-room {
              min-height: 70px;
              width: 100%;
              border-right: 1px dashed #bcbcbc;
              border-top: none;
              border-bottom: 1px dashed #bcbcbc;
              cursor: pointer;
              box-sizing: border-box;
              user-select: none;
              padding: 2px;

              .room {
                &-info {
                  height: 100%;
                  background: #00a8cd;
                  border-radius: 5px;
                  color: #fff;
                  font-weight: bold;
                  font-size: 14px;
                  display: flex;
                  flex-direction: column;
                  justify-content: space-between;

                  &-name {
                    padding: 0 5px 0 15px;
                  }

                  &-type {
                    padding-top: 6px;
                    padding-left: 10px;
                    font-size: 13px;

                    .ico {
                      width: 13px;
                    }
                  }

                  &-capacity {
                    padding-bottom: 6px;
                    padding-right: 10px;
                    text-align: right;

                    .ico {
                      width: 13px;
                    }
                  }
                }
              }
            }
          }

          &-reservation {
            background-color: rgba(0, 168, 205, 0.2);
            position: absolute;
            top: 0;
            bottom: 0;
            width: 100px;
            border-radius: 5px;
            color: #000;
            font-size: 14px;
            cursor: pointer;
            transition: background 0.5s, min-width 0.3s ease, width 0.3s ease;
            user-select: none;
            display: flex;
            overflow: hidden;

            &-dragging {
              cursor: grabbing;
            }

            &-dragging {
              background-color: rgba(0, 168, 205, 0.7);
              pointer-events: none;
              opacity: 0.4;
            }

            &-clone {
              box-shadow: 0 20px 25px -5px rgba(0, 0, 0, 0.1),
                0 10px 10px -5px rgba(0, 0, 0, 0.04);
              transition: background 0.5s !important;
            }

            &-resizing {
              transition: background 0.5s !important;
            }

            &:hover {
              background-color: rgba(0, 168, 205, 0.7);
            }

            &-content {
              height: 100%;
              flex: 1;
              display: flex;
              flex-direction: column;
              position: relative;
              justify-content: center;
              color: #223345;

              &-room {
                &-time {
                  position: absolute;
                  right: 5px;
                  top: 8px;
                  font-weight: 600;
                  font-size: 12px;
                  line-height: 12px;
                  text-align: right;
                  color: #223345;
                  font-variant: tabular-nums;
                  display: -webkit-box;
                  -webkit-line-clamp: 1;
                  -webkit-box-orient: vertical;
                  overflow: hidden;
                }

                &-name {
                  font-weight: bold;
                  font-size: 14px;
                  line-height: 13px;
                  color: #223345;
                  margin-top: 9px;
                  display: -webkit-box;
                  -webkit-line-clamp: 1;
                  -webkit-box-orient: vertical;
                  overflow: hidden;
                }

                &-area {
                  font-weight: 500;
                  font-size: 11px;
                  line-height: 11px;
                  color: #223345;
                  margin-top: 2px;
                  display: -webkit-box;
                  -webkit-line-clamp: 1;
                  -webkit-box-orient: vertical;
                  overflow: hidden;
                }

                &-customer {
                  font-weight: 500;
                  font-size: 11px;
                  line-height: 11px;
                  color: #223345;
                  margin-top: 2px;
                  display: -webkit-box;
                  -webkit-line-clamp: 1;
                  -webkit-box-orient: vertical;
                  overflow: hidden;
                }
              }
            }

            &-resize {
              flex: 0 0 20px;
              cursor: ew-resize;
            }
          }
        }
      }
    }
  }
}
</style>
