<!-- AllRooms.vue -->
<template>
<div>
    <v-container>
        <v-row>
            <v-col cols="12" class="text-center">
                <h1>ยินดีต้อนรับสู่ ชมรม SMC</h1>
            </v-col>
        </v-row>
        <v-row>
            <v-col cols="12" class="text-center">
                <h3>ภาพห้องซ้อมดนตรีทั้งหมด</h3>
            </v-col>
        </v-row>
        <v-row>
            <v-col cols="12" sm="6" md="4">
                <v-card min-height="300px" min-weight="300px" class="text-center align-center" style="display: flex; flex-direction: column; align-items: center;">
                    <v-img src="..\src\assets\picture\ห้องซ้อม smc 1.jpg" alt=" "></v-img>
                    <v-card-title class="text-center">ห้องซ้อมดนตรี 1</v-card-title>
                    <v-btn color="primary" @click="openBookingPopup('ห้องซ้อมดนตรี 1', '1')"> จอง </v-btn>
                </v-card>
            </v-col>
            <v-col cols="12" sm="6" md="4">
                <v-card min-height="300px" min-weight="300px" class="text-center align-center" style="display: flex; flex-direction: column; align-items: center;">
                    <v-img src="..\src\assets\picture\ห้องซ้อม smc 2.jpg" alt=" "></v-img>
                    <v-card-title>ห้องซ้อมดนตรี 2</v-card-title>
                    <v-btn color="primary" @click="openBookingPopup('ห้องซ้อมดนตรี 2', '2')"> จอง </v-btn>
                </v-card>
            </v-col>
            <v-col cols="12" sm="6" md="4">
                <v-card min-height="300px" min-weight="300px" class="text-center align-center" style="display: flex; flex-direction: column; align-items: center;">
                    <v-img src="..\src\assets\picture\ห้องซ้อมดนตรี3.jpg" alt=" "></v-img>
                    <v-card-title>ห้องซ้อมดนตรี 3</v-card-title>
                    <v-btn color="primary" @click="openBookingPopup('ห้องซ้อมดนตรี 3', '3')"> จอง </v-btn>
                </v-card>
            </v-col>
        </v-row>
    </v-container>
    <!-- ป๊อปอัพการจอง -->
    <v-dialog v-model="isBookingPopupVisible" max-width="500px">
        <v-card>
            <v-card-title>{{bookingData.roomName}}</v-card-title>
            <v-card-text>
                <v-form ref="bookingForm">
                    <v-date-picker v-model="bookingData.bookingDate" label="วันที่" required :min="minDate"></v-date-picker>
                    <v-select v-model="bookingData.bookingTime" :items="timeSlots" label="ช่วงเวลา" required></v-select>
                </v-form>
            </v-card-text>
            <v-card-actions>
                <v-btn color="primary" @click="bookRoom"> จอง </v-btn>
                <v-btn color="red" @click="isBookingPopupVisible = false"> ยกเลิก </v-btn>
            </v-card-actions>
        </v-card>
    </v-dialog>
</div>
</template>

<!-- eslint-disable camelcase -->

<script>
import EventBus from './EventBus'
export default {
  name: 'AllRooms',

  data () {
    return {
      studentId: '',
      isBookingPopupVisible: false,
      isBookingSuccess: false,
      bookingData: {
        roomName: '',
        room_id: '',
        bookingDate: null,
        bookingTime: null
      },
      minDate: new Date().toISOString().substr(0, 10),
      timeSlots: [
        '16:00 - 17:30',
        '17:30 - 19:00',
        '19:00 - 20:30',
        '20:30 - 22:00'
      ]
    }
  },

  methods: {
    openBookingPopup (roomName, room_id) {
      this.isBookingPopupVisible = true
      this.bookingData.roomName = roomName
      this.bookingData.room_id = room_id
      const auth = JSON.parse(localStorage.getItem('auth'))
      if (auth) {
        this.studentId = auth.studentId
      }
    },

    bookRoom () {
      if (this.studentId && this.isFormValid) {
        this.bookingData.studentId = this.studentId

        const reservation = {
          room_id: this.bookingData.room_id,
          studentId: this.bookingData.studentId,
          reserve_date: this.bookingData.bookingDate,
          reserve_time: this.bookingData.bookingTime
        }

        this.axios
          .post('http://localhost:9001/reserveroom', reservation)
          .then(response => {
            console.log('Reservation response:', response.data)
            alert('จองห้องสำเร็จ')
            this.isBookingPopupVisible = false
            EventBus.$emit('booking-made', this.bookingData)
            this.clearBookingData()
          })
          .catch(error => {
            console.error('Reservation error:', error)
          })
      } else {
        alert('กรุณากรอกข้อมูลให้ถูกต้องตามกฎและครบถ้วน')
      }
    },

    clearBookingData () {
      this.bookingData.bookingDate = null
      this.bookingData.bookingTime = null
      this.isBookingPopupVisible = false
    }
  },

  computed: {
    isFormValid () {
      return (
        this.$refs.bookingForm.validate()
      )
    }
  }
}
</script>
