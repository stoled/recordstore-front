<template>
  <header>
    <b-nav align="end">
      <b-nav-item to="/" v-if="!signedIn()">Sign in</b-nav-item>
      <b-nav-item to="/signup" v-if="!signedIn()">Sign Up</b-nav-item>
      <b-nav-item to="/records" v-if="signedIn()">Records</b-nav-item>
      <b-nav-item to="/artists" v-if="signedIn()">Artists</b-nav-item>
      <a href="#" @click.prevent="signOut" v-if="signedIn()">Sign out</a>
    </b-nav>
  </header>
</template>

<script>
export default {
  name: 'Header',
  created () {
    this.signedIn()
  },
  methods: {
    setError (error, text) {
      this.error = (error.response && error.response.data && error.response.data.error) || text
    },
    signedIn () {
      return localStorage.signedIn
    },
    signOut () {
      this.$http.secured.delete('/signin')
        .then(response => {
          delete localStorage.csrf
          delete localStorage.signedIn
          this.$router.replace('/')
        })
        .catch(error => this.setError(error, 'Cannot sign out'))
    }
  }
}
</script>