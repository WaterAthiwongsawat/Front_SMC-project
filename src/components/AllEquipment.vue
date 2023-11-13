<template>
  <div>
    <v-container  style="margin-bottom: 80px;">
      <v-row>
        <v-col
          v-for="(item, index) in items"
          :key="index"
          cols="6"
        >
          <v-card
            :loading="loading"
            class="mx-auto my-4 d-flex flex-column card-hover"
            max-width="374"
          >
            <template slot="progress">
              <v-progress-linear
                color="deep-purple"
                height="10"
                indeterminate
              ></v-progress-linear>
            </template>

            <!-- <v-img
              height="250"
              :src="require(`@/assets/picture/${item.eqUrl}`)"
            ></v-img> -->

            <v-img
            height="250"
            :src="item.eqUrl"
            ></v-img>

            <v-card-title>{{ item.eqName }}</v-card-title>

            <v-card-text>
              <div>จำนวนคงเหลือ {{ item.remaining }} ตัว</div>
            </v-card-text>

            <v-card-actions>
              <v-btn
                color="deep-purple lighten-2"
                outlined
                class="mt-auto"
                @click="openBookingPopup(item.eqName,item.eqId)"
                style="width: 100%;"
                elevation="1"
              >
                จอง
              </v-btn>
            </v-card-actions>
          </v-card>
        </v-col>
      </v-row>
    </v-container>
    <v-dialog v-model="isBookingPopupVisible" max-width="500px">
    <v-card>
      <v-card-title>{{bookingData.eqName}}</v-card-title>
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
<!-- ป๊อปอัพการจอง -->
  </template>

<script>
import axios from 'axios'
// import EventBus from './EventBus'
export default {
  name: 'borrowEq',
  data () {
    return {
      loading: true,
      items: [],
      studentId: '',
      isBookingPopupVisible: false,
      bookingData: {
        eqName: '',
        bookingDate: null,
        bookingTime: ''
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

  created () {
    this.fetchEquipment()
  },

  methods: {
    fetchEquipment () {
      axios.get('http://localhost:9001/Equipment')
        .then((response) => {
          this.items = response.data
          this.loading = false
        })
        .catch((error) => {
          console.error('Error fetching Equipment data:', error)
          this.loading = false
        })
    },
    openBookingPopup (eqName, eqId) {
      this.isBookingPopupVisible = true
      this.bookingData.eqName = eqName
      this.bookingData.eqId = eqId
      const auth = JSON.parse(localStorage.getItem('auth'))
      if (auth) {
        this.studentId = auth.studentId
      }
    },

    bookRoom () {
      if (this.studentId && this.isFormValid) {
        this.bookingData.studentId = this.studentId

        const reservation = {
          eqId: this.bookingData.eqId,
          studentId: this.bookingData.studentId,
          borrow_date: this.bookingData.bookingDate,
          borrow_time: this.bookingData.bookingTime,
          borrow: 1
        }

        console.log(reservation)

        axios
          .post('http://localhost:9001/borroeEq', reservation)
          .then((response) => {
            console.log('Reservation response:', response.data)
            alert('จองอุปกรณ์สำเร็จ')
            // Call the updateRemaining API here
            return axios.put(`http://localhost:9001/updateRemaining/${this.bookingData.eqId}`)
          })
          .then((updateResponse) => {
            console.log('Update Remaining response:', updateResponse.data)
            this.isBookingPopupVisible = false
            // EventBus.$emit('booking-made', this.bookingData); // Uncomment if needed
            this.clearBookingData()
            window.location.reload()
          })
          .catch((error) => {
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

<style scoped>
  .card-hover:hover {
    box-shadow: 0 0 10px rgba(0, 0, 0, 0.5); /* Add a shadow when hovering */
  }
  </style>
