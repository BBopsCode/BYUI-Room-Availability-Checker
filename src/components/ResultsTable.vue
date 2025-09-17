<template>
  <v-card elevation="2" className="mt-6 pa-4">
    <v-card-title>Available Rooms</v-card-title>
    <v-data-table
        :items="results"
        :headers="headers"
        dense
        class="elevation-1"
    />
  </v-card>
</template>

<script setup>
import {defineProps} from 'vue'

const props = defineProps({
  results: {
    type: Array,
    default: () => []
  }
})

const headers = [
  {text: 'Room Number', value: 'room'},
  {text: 'Building', value: 'building'},
  {text: 'Start Time', value: 'start_time'},
  {text: 'End Time', value: 'end_time'},
]
</script>
