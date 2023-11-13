<template>
    <div>
      <v-data-table :headers="headers" :items="EquipmentItem" class="elevation-1">
          <template v-slot:top>
              <v-toolbar flat>
                  <v-toolbar-title>หน้าจัดการข้อมูลการยืม-คืน</v-toolbar-title>
                  <v-divider class="mx-4" inset vertical></v-divider>
                  <v-spacer></v-spacer>
              </v-toolbar>
          </template>
          <template v-slot:[`item.status`]="{ item }">
            <span v-if="item.borrow === true">ทำการยืมสำเร็จ</span>
          <span v-else-if="item.borrow === false">คืนสำเร็จ</span>
        </template>
        <template v-slot:[`item.actions`]="{ item }">
            <v-btn
                small
                outlined
                @click="deleteItem(item)"
                color="red"
                class="ml-2"
                :disabled="!item.borrow"
            >
                คืนอุปกรณ์
            </v-btn>
        </template>
          <template v-slot:no-data>
              ไม่มีรายการ การยืม
          </template>
      </v-data-table>
      <v-dialog v-model="dialogDelete" max-width="500px">
          <v-card>
              <v-card-title class="text-h5">คุณต้องการคืนอุปกรณ์นี้ใช่ หรือ ไม่?</v-card-title>
              <v-card-actions>
                  <v-spacer></v-spacer>
                  <v-btn color="blue darken-1" text @click="closeDelete">ยกเลิก</v-btn>
                  <v-btn color="blue darken-1" text @click="deleteItemConfirm">ตกลง</v-btn>
                  <v-spacer></v-spacer>
              </v-card-actions>
          </v-card>
      </v-dialog>
    </div>
  </template>

<script>
import EventBus from './EventBus'
import axios from 'axios'
/* eslint-disable */
  export default {
      data: () => ({
          equipmentOptions: ['Microphone', 'Guitar Bass', 'Electric Guitar', 'Keyboard', 'Stand Note', 'Drum Kit'],
          minDate: new Date().toISOString().substr(0, 10),
          timeSlots: [
        '17.00 - 18.20',
        '18:20 - 19:40',
        '19.40 - 21.00',
        '21:00 - 22:20',
        '22.20 - 23.40'

          ],
          Equipment_id: '',
          room_id: '',
          studentId: '',
          reserve_date: '',
          reserve_time: '',
          dialogCreate: false,
          dialogDelete: false,
          headers: [{
                  text: 'ลำดับการยืม-คืน',
                  align: 'start',
                  sortable: false,
                  value: 'borrow_id'
              },
              {
                  text: 'รหัสนักศึกษาผู้ที่ทำการยืม-คืน',
                  align: 'start',
                  sortable: false,
                  value: 'studentId'
              },
              {
                  text: 'รหัสอุปกรณ์',
                  value: 'eqId'
              },
              {
                  text: 'ชื่ออุปกรณ์',
                  value: 'EqName'
              },
              {
                  text: 'วันที่ทำการยืม',
                  value: 'borrow_date'
              },
              {
                  text: 'สถานะการยืม',
                  value: 'status'
              },
              {
                  text: 'จัดการ',
                  value: 'actions',
                  sortable: false
              }
          ],
          EquipmentItem: [],
          editedIndex: -1,
          editedItem: {},
          defaultItem: {},
          formTitle: '',
          idEquipment: '',
          idforDelete: '',
          idEq: '',
          userData: {}
      }),
  
      watch: {
          dialog(val) {
              val || this.close()
          },
          dialogDelete(val) {
              val || this.closeDelete()
          }
      },
  
      created() {
        console.log('Component created')
        const authData = localStorage.getItem('auth')
        if (authData) {
        this.userData = JSON.parse(authData)
        console.log('User data:', this.userData)
        this.userId = this.userData.userId
        console.log('UserId: ', this.userId)
        console.log(this.userData.studentId)
        }
  
        this.initialize()
        EventBus.$on('booking-made', (bookingData) => {
            console.log('Booking made:', bookingData);
        })
      },
  
      methods: {
          async initialize() {
              this.EquipmentItem = []
              try {
                  const data = await this.axios.get(`http://localhost:9001/borrowHistory/${this.userData.studentId}`)
                  this.EquipmentItem = data.data
                  for (const item of this.EquipmentItem){
                    await this.fetchEqName(item.eqId, item)
                  }
                  console.log('data Equipment ====>', data)
              } catch (error) {
  
              }
          },
          async fetchEqName (eqId, item){
            try {
                const response = await axios.get(`http://localhost:9001/Equipment/${eqId}`)
                const eqName = response.data.eqName

                // Update the 'eqName' property for the item
                this.$set(item, 'EqName', eqName)

            } catch (error) {
                console.error('Error fetching eqName name:', error)
            }
          },
  
          deleteItem(item) {
              this.idforDelete = item.borrow_id
              this.dialogDelete = true
              this.idEq = item.eqId
          },
  
          async deleteItemConfirm() {
    try {
        // Update isBorrow field to false
        await this.axios.put('http://localhost:9001/reBorrow/' + this.idforDelete);

        // Assuming 'reupdateRemaining' is the endpoint to update another field (replace it with the correct endpoint)
        await this.axios.put('http://localhost:9001/reupdateRemaining/' + this.idEq);

        this.initialize();
    } catch (error) {
        console.log(error.message);
    }
    this.closeDelete();
},

  
          closeDelete() {
              this.dialogDelete = false
              this.$nextTick(() => {
                  this.editedItem = Object.assign({}, this.defaultItem)
                  this.editedIndex = -1
              })
          },
      },
      computed: {
        statusMessage() {
            return (borrow) => {
            return borrow === 1 ? 'ทำการยืมสำเร็จ' : 'คืนสำเร็จ';
      }
    },
  },
  }
  </script>
  
  <style>
  </style>
  