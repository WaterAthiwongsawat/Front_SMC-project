<template>
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
                @click="seeDetails(index)"
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
  </template>

<script>
import axios from 'axios'
export default {
  name: 'borrowEq',
  data: () => ({
    loading: true,
    items: []
  }),

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

    seeDetails (index) {
      if (typeof index === 'number' && index >= 0 && index < this.items.length) {
        console.log('Viewing details of item at index:', index)
        this.$router.push({ path: `/rooms/${index}` })
      } else {
        console.error('Invalid index:', index)
      }
    }

  }
}
</script>

  <style scoped>
  .card-hover:hover {
    box-shadow: 0 0 10px rgba(0, 0, 0, 0.5); /* Add a shadow when hovering */
  }
  </style>
