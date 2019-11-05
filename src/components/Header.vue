<template>
  <header>
    <b-nav justified>
      <b-nav-item to="/records">Records</b-nav-item>
      <b-nav-item to="/artists">Artists</b-nav-item>
      <b-nav-item to="/">Sign in</b-nav-item>
      <b-nav-item to="/signup">Sign Up</b-nav-item>
      <b-nav-item to="/" @click.prevent="signOut">Sign out</b-nav-item>
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