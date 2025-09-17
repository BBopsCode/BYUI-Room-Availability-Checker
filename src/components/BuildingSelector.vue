<template>
  <v-card elevation="8" class="pa-6" max-width="400">
    <v-card-title>Select a Building</v-card-title>

    <v-card-text>
      <v-select
          v-model="selectedBuilding"
          :items="locations"
          label="Building"
          outlined
          dense
      />
    </v-card-text>

    <v-card-actions>
      <v-btn color="primary" @click="confirmSelection">Confirm</v-btn>
    </v-card-actions>
  </v-card>
</template>

<script setup>
import { ref } from 'vue'
import rooms from '../../public/university_schedule.json'

const locations = [
  'Smith', 'Kimball', 'Ricks', 'STC', 'Hinckley', 'McKay', 'Hinckley Chapel',
  'Spori', 'BEN', 'Austin', 'Taylor', 'Taylor Chapel', 'McKay Library',
  'Rigby', 'Clarke', 'Benson', 'Romney', 'MC', 'Hart', 'AUS', 'Snow'
]

const selectedBuilding = ref(null)
const emit = defineEmits(['search'])

const confirmSelection = () => {
  if (!selectedBuilding.value) {
    alert('Please select a building first')
    return
  }

  const now = new Date()
  const currentMinutes = now.getHours() * 60 + now.getMinutes()
  const today = now.getDay() // Sunday=0
  const dayMap = ['S', 'M', 'T', 'W', 'R', 'F', 'Sa']

  // Filter rooms in selected building and for today
  const buildingRooms = rooms.filter(room => {
    if (room.building !== selectedBuilding.value) return false
    if (!room.days.includes(dayMap[today])) return false

    const startDate = new Date(room.start_date)
    const endDate = new Date(room.end_date)
    if (now < startDate || now > endDate) return false

    return true
  })

  // Calculate how many minutes each room is free
  const availableRooms = buildingRooms.map(room => {
    const [startH, startM] = room.start_time.split(':').map(Number)
    const [endH, endM] = room.end_time.split(':').map(Number)
    const startMinutes = startH * 60 + startM
    const endMinutes = endH * 60 + endM

    let freeMinutes = 0
    if (currentMinutes < startMinutes) {
      freeMinutes = startMinutes - currentMinutes
    } else if (currentMinutes >= endMinutes) {
      // room is free until next reservation (or end of day)
      freeMinutes = 24*60 - currentMinutes // optional
    } else {
      freeMinutes = 0 // currently occupied
    }

    return {
      building: room.building,
      room: room.room,
      freeMinutes
    }
  }).filter(r => r.freeMinutes > 0) // only include rooms that are actually free

  console.log('Available rooms with free time:', availableRooms)
  emit('search', availableRooms)
}
</script>
