<template>
  <v-container>
    <h2>Room Availability Checker</h2>

    <v-row class="mb-4">
      <v-col cols="12" md="6">
        <v-select
            v-model="selectedBuilding"
            :items="buildings"
            label="Select Building"
            outlined
        ></v-select>
      </v-col>
      <v-col cols="12" md="6">
        <v-switch
            v-model="useRealTime"
            label="Use real time & day"
            color="primary"
            @change="findAvailableRooms"
        ></v-switch>

        <div v-if="!useRealTime">
          <v-text-field
              v-model="selectedTime"
              label="Select Time (24-hour format)"
              type="time"
              outlined
              dense
              @input="findAvailableRooms"
              hint="Format: HH:MM (e.g., 13:34)"
              class="mb-3"
          ></v-text-field>

          <v-select
              v-model="selectedDay"
              :items="dayOptions"
              label="Select Day"
              outlined
              dense
              @change="findAvailableRooms"
          ></v-select>
        </div>
      </v-col>
    </v-row>

    <!-- Debug Info -->
    <v-card class="mb-4" color="blue-grey lighten-5">
      <v-card-text>
        <h4>Debug Info:</h4>
        <p><strong>Data loaded:</strong> {{ scheduleData.length }} sessions</p>
        <p><strong>Current time:</strong> {{ displayTime }}</p>
        <p><strong>Current day:</strong> {{ displayDay }}</p>
        <p><strong>Selected building:</strong> {{ selectedBuilding || 'None' }}</p>
        <p><strong>Available rooms:</strong> {{ availableRooms.length }}</p>
      </v-card-text>
    </v-card>

    <!-- Available Rooms -->
    <div v-if="selectedBuilding">
      <h3>Available Rooms in {{ selectedBuilding }}</h3>
      <v-select v-if="availableRooms.length > 0"
          v-model="selectedFilter"
          :items="filterOptions"
          label="Filter"
          dense
          outlined
          style="width: 200px;"
      ></v-select>


      <v-card v-if="availableRooms.length > 0">
        <v-list>
          <v-list-item v-for="room in displayedRooms" :key="room.room">
            <v-list-item-title>
              Room {{ room.room }}
            </v-list-item-title>
            <v-list-item-subtitle>
              Free for {{ room.freeMinutes }} minutes
            </v-list-item-subtitle>
            <template v-slot:append>
              <v-chip :color="room.freeMinutes > 60 ? 'green' : 'orange'">
                {{ room.freeMinutes }}min
              </v-chip>
            </template>
          </v-list-item>
        </v-list>
      </v-card>

      <v-alert v-else type="info">
        No available rooms found in {{ selectedBuilding }} right now.
      </v-alert>
    </div>
  </v-container>
</template>

<script>
export default {
  name: 'RoomAvailabilityChecker',
  data() {
    return {
      selectedBuilding: null,
      selectedFilter: 'Free Time',
      availableRooms: [],
      scheduleData: [],
      buildings: ['Smith', 'Kimball', 'Ricks', 'STC', 'Hinckley', 'McKay', 'Spori', 'Austin', 'Taylor', 'Benson', 'Romney', 'Clarke', 'Hart', 'Snow'],
      filterOptions: ['Room Number', 'Free Time'],
      useRealTime: false, // Default to simulated time for testing
      selectedTime: '13:37', // Default test time
      selectedDay: 'T', // Default to Tuesday
      dayOptions: [
        { title: 'Monday', value: 'M' },
        { title: 'Tuesday', value: 'T' },
        { title: 'Wednesday', value: 'W' },
        { title: 'Thursday', value: 'R' },
        { title: 'Friday', value: 'F' },
        { title: 'Saturday', value: 'S' },
        { title: 'Sunday', value: 'U' }
      ]
    }
  },
  computed: {
    displayedRooms() {
      if (this.selectedFilter === 'Free Time') {
        return this.availableRooms.sort((a, b) => b.freeMinutes - a.freeMinutes);
      } else if (this.selectedFilter === 'Room Number') {
        return this.availableRooms.sort((a, b) => a.room.localeCompare(b.room));
      }
      return this.availableRooms;
    },
    displayTime() {
      if (this.useRealTime) {
        const now = new Date();
        const hours = now.getHours().toString().padStart(2, '0');
        const minutes = now.getMinutes().toString().padStart(2, '0');
        return `${hours}:${minutes} (real time)`;
      } else {
        return `${this.selectedTime} (selected)`;
      }
    },
    displayDay() {
      if (this.useRealTime) {
        const days = ['Sunday', 'Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday', 'Saturday'];
        const dayAbbrev = ['U', 'M', 'T', 'W', 'R', 'F', 'S'];
        const dayIndex = new Date().getDay();
        return `${days[dayIndex]} (${dayAbbrev[dayIndex]}) - real`;
      } else {
        const dayNames = {
          'M': 'Monday',
          'T': 'Tuesday',
          'W': 'Wednesday',
          'R': 'Thursday',
          'F': 'Friday',
          'S': 'Saturday',
          'U': 'Sunday'
        };
        return `${dayNames[this.selectedDay]} (${this.selectedDay}) - selected`;
      }
    },
    currentMinutes() {
      if (this.useRealTime) {
        const now = new Date();
        return now.getHours() * 60 + now.getMinutes();
      } else {
        return this.timeToMinutes(this.selectedTime);
      }
    },
    currentDay() {
      if (this.useRealTime) {
        const dayAbbrev = ['U', 'M', 'T', 'W', 'R', 'F', 'S'];
        return dayAbbrev[new Date().getDay()];
      } else {
        return this.selectedDay;
      }
    }
  },
  watch: {
    selectedBuilding() {
      console.log('Building selected:', this.selectedBuilding);
      this.findAvailableRooms();
    }
  },
  async mounted() {
    console.log('Component mounted');
    await this.loadData();
  },
  methods: {
    async loadData() {
      try {
        console.log('Loading data...');
        const response = await fetch('/university_schedule.json');
        this.scheduleData = await response.json();
        console.log('Data loaded:', this.scheduleData.length, 'sessions');
      } catch (error) {
        console.error('Error loading data:', error);
      }
    },

    findAvailableRooms() {
      console.log('\n=== Finding available rooms ===');

      if (!this.selectedBuilding || !this.scheduleData.length) {
        this.availableRooms = [];
        return;
      }

      const currentMinutes = this.currentMinutes;
      const currentDay = this.currentDay;

      console.log('Current time:', currentMinutes, `minutes (${this.minutesToTime(currentMinutes)})`);
      console.log('Current day:', currentDay);
      console.log('Using real time:', this.useRealTime);
      console.log('Looking for building:', this.selectedBuilding);

      // Step 1: Find all rooms in this building
      const allRooms = new Set();
      this.scheduleData.forEach(session => {
        if (session.building === this.selectedBuilding && session.room) {
          allRooms.add(session.room);
        }
      });

      console.log('All rooms found:', Array.from(allRooms));

      // Step 2: Check each room
      this.availableRooms = [];

      allRooms.forEach(roomNumber => {
        console.log(`\nChecking room ${roomNumber}:`);

        // Get today's classes for this room
        const todaysClasses = this.scheduleData.filter(session => {
          return session.building === this.selectedBuilding &&
              session.room === roomNumber &&
              session.days && session.days.includes(currentDay) &&
              this.isDateInRange(session.start_date, session.end_date);
        });

        console.log(`  Found ${todaysClasses.length} classes today`);

        // Check if any class is happening now
        let isOccupied = false;
        let nextClassTime = null;

        for (const cls of todaysClasses) {
          const startMinutes = this.timeToMinutes(cls.start_time);
          const endMinutes = this.timeToMinutes(cls.end_time);

          console.log(`  Class: ${cls.start_time}-${cls.end_time} (${startMinutes}-${endMinutes})`);

          // Check if currently occupied
          if (currentMinutes >= startMinutes && currentMinutes < endMinutes) {
            console.log(`    OCCUPIED NOW`);
            isOccupied = true;
            break;
          }

          // Check for next class
          if (startMinutes > currentMinutes) {
            if (!nextClassTime || startMinutes < this.timeToMinutes(nextClassTime)) {
              nextClassTime = cls.start_time;
            }
          }
        }

        if (!isOccupied) {
          let freeMinutes;
          if (nextClassTime) {
            freeMinutes = this.timeToMinutes(nextClassTime) - currentMinutes;
            console.log(`  FREE until ${nextClassTime} (${freeMinutes} minutes)`);
          } else {
            freeMinutes = (23 * 60 + 59) - currentMinutes; // Until end of day
            console.log(`  FREE for rest of day (${freeMinutes} minutes)`);
          }

          this.availableRooms.push({
            room: roomNumber,
            freeMinutes: freeMinutes,
            nextClass: nextClassTime
          });
        }
      });

      console.log('Available rooms:', this.availableRooms.length);
      this.availableRooms.sort((a, b) => b.freeMinutes - a.freeMinutes);
    },

    timeToMinutes(timeStr) {
      const [hours, minutes] = timeStr.split(':').map(Number);
      return hours * 60 + minutes;
    },

    minutesToTime(minutes) {
      const hours = Math.floor(minutes / 60);
      const mins = minutes % 60;
      return `${hours.toString().padStart(2, '0')}:${mins.toString().padStart(2, '0')}`;
    },

    isDateInRange(startDate, endDate) {
      const today = new Date();
      const start = new Date(startDate);
      const end = new Date(endDate);
      return today >= start && today <= end;
    }
  }
}
</script>

<style scoped>
.v-list-item {
  border-bottom: 1px solid #eee;
}
</style>