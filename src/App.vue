<template>
  <v-app>
    <v-main>
      <RoomAvailabilityChecker />
    </v-main>
  </v-app>
</template>

<script>
import RoomAvailabilityChecker from './components/RoomAvailabilityChecker.vue'

export default {
  name: 'App',
  components: {
    RoomAvailabilityChecker
  }
}
</script>