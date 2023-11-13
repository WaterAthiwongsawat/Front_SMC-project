<template>
    <v-container>
        <v-row>
            <v-col cols="6" sm="4">
                <h1>Profile Page</h1>
                <div v-if="userData">
                    <p><strong>Student Id:</strong> {{ userData.studentId }}</p>
                    <p><strong>User Name:</strong> {{ userData.name }} {{ userData.surname }}</p>
                    <p><strong>User Email:</strong> {{ userData.email }}</p>
                    <p><strong>User Phone:</strong> {{ userData.telephonenumber }}</p>
                </div>
            </v-col>
        </v-row>
    </v-container>

</template>

<script>
export default {
  created () {
    console.log('Component created')
    const authData = localStorage.getItem('auth')
    if (authData) {
      this.userData = JSON.parse(authData)
      console.log('User data:', this.userData)
      this.userId = this.userData.userId
      console.log('UserId: ', this.userId)
    }
    this.initialize()
  }

}
</script>

<style>

</style>
