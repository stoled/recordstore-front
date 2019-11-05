<template>
  <div>
    <div v-if="error">{{ error }}</div>
    <h3 >Add a new record</h3>

    <form @submit.prevent="addRecord">
      <div>
        <label for="record_title">Title</label>
        <input
          type="text"
          id="record_title"
          autofocus
          autocomplete="off"
          placeholder="Type a record name"
          v-model="newRecord.title">
      </div>

      <div>
        <label for="record_year">Year</label>
        <input
          type="text"
          id="record_year"
          autofocus
          autocomplete="off"
          placeholder="Year"
          v-model="newRecord.year">
      </div>

      <div>
        <label for="artist">Artist</label>
        <select id="artist" v-model="newRecord.artist">
          <option disabled value="">Select an artist</option>
          <option :value="artist.id" v-for="artist in artists" :key="artist.id">{{ artist.name }}</option>
        </select>
        <p>Don't see an artist? <router-link to="/artists">Create one</router-link></p>
      </div>

      <input type="submit" value="Add Record">
    </form>

    <hr>

    <ul>
      <li v-for="record in records" :key="record.id" :record="record">
        <div>
          <div>
          <p>
            {{ record.title }} &mdash; {{ record.year }}
          </p>
          <p>{{ getArtist(record) }}</p>
        </div>

        <button @click.prevent="editRecord(record)">Edit</button>

        <button @click.prevent="removeRecord(record)">Delete</button>
        </div>

        <div v-if="record == editedRecord">
          <form action="" @submit.prevent="updateRecord(record)">
            <div>

              <div>
                <label>Title</label>
                <input v-model="record.title">
              </div>

              <div>
                <label>Year</label>
                <input v-model="record.year">
              </div>

              <div>
                 <select id="artist_update" v-model="record.artist">
                    <option disabled value="">Select an artist</option>
                    <option :value="artist.id" v-for="artist in artists" :key="artist.id">{{ artist.name }}</option>
                  </select>
              </div>

              <input type="submit" value="Update">
            </div>
          </form>
        </div>
      </li>
    </ul>
  </div>
</template>

<script>
export default {
  name: 'Records',
  data () {
    return {
      artists: [],
      records: [],
      newRecord: [],
      error: '',
      editedRecord: ''
    }
  },
  created () {
    if (!localStorage.signedIn) {
      this.$router.replace('/')
    } else {
      this.$http.secured.get('/api/v1/records')
        .then(response => { this.records = response.data })
        .catch(error => this.setError(error, 'Something went wrong'))
      this.$http.secured.get('/api/v1/artists')
        .then(response => { this.artists = response.data })
        .catch(error => this.setError(error, 'Something went wrong'))
    }
  },
  methods: {
    setError (error, text) {
      this.error = (error.response && error.response.data && error.response.data.error) || text
    },
    getArtist (record) {
      const recordArtistValues = this.artists.filter(artist => artist.id === record.artist_id)
      let artist
      recordArtistValues.forEach(function (element) {
        artist = element.name
      })
      return artist
    },
    addRecord () {
      const value = this.newRecord
      if (!value) {
        return
      }
      this.$http.secured.post('/api/v1/records/', { record: { title: this.newRecord.title, year: this.newRecord.year, artist_id: this.newRecord.artist } })
        .then(response => {
          this.records.push(response.data)
          this.newRecord = ''
        })
        .catch(error => this.setError(error, 'Cannot create record'))
    },
    removeRecord (record) {
      this.$http.secured.delete(`/api/v1/records/${record.id}`)
        .then(response => {
          this.records.splice(this.records.indexOf(record), 1)
        })
        .catch(error => this.setError(error, 'Cannot delete record'))
    },
    editRecord (record) {
      this.editedRecord = record
    },
    updateRecord (record) {
      this.editedRecord = ''
      this.$http.secured.patch(`/api/v1/records/${record.id}`, { record: { title: record.title, year: record.year, artist_id: record.artist } })
        .catch(error => this.setError(error, 'Cannot update record'))
    }
  }
}
</script>