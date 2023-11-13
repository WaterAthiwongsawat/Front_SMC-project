<template>
    <div>
      <v-card>
      </v-card>
      <v-btn color="error" @click="logout" block>
        ออกจากระบบ
      </v-btn>
    </div>
  </template>

<script>
export default {

  methods: {

    logout () {
      localStorage.removeItem('auth')
      this.$router
        .push({ path: '/' })
        .then(() => {
          window.location.reload()
        })
        .catch(() => {})

      console.log('Logout successful')
      alert('ออกจากระบบสำเร็จ')
    }
  }
}
</script>

  <style>
  </style>
