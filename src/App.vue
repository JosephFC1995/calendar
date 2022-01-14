<template>
  <div id="app">
    <CalendarGrid
      :rooms="rooms"
      :dateStart="{ hour: 6, minute: 0 }"
      :minHeight="100"
      :widthCell="120"
      :reservations="reservations"
      :reservationNew.sync="reservationNew"
      @reservationMove="onReservationMove"
      @reservationResize="onReservationResize"
    />
    <pre>
      {{ reservationNew }}
    </pre>
  </div>
</template>

<script lang="ts">
import { Component, Vue, Watch } from "vue-property-decorator";
import CalendarGrid from "./components/CalendarGrid.vue";
import roomsHelper from "./helper/room.json";
import reservationHelper from "./helper/reservation.json";

@Component({
  components: { CalendarGrid },
})
export default class App extends Vue {
  rooms: Array<any> = roomsHelper;
  reservations: any = reservationHelper;
  reservationNew: any = null;

  reservationEdit: any = null;
  reservationResize: any = null;

  onReservationMove(reservation: any): void {
    alert("RESERVATION MOVE");
    console.log("RESERVATION MOVE", reservation);
    this.reservationEdit = reservation;
  }

  onReservationResize(reservation: any): void {
    alert("RESERVATION RESIZE");
    console.log("RESERVATION RESIZE", reservation);
    this.reservationResize = reservation;
  }
}
</script>

<style lang="scss" scoped>
* {
  outline: none;
  margin: 0;

  h1 {
    font-size: 1rem;
  }
}

#app {
  background-color: #f3f7f9;
  padding: 2rem;
}
</style>

<style>
body {
  margin: 0;
}
</style>
