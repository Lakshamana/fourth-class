<template>
  <div id="index">
    <div class="container">
      <div class="row justify-content-center">
        <h1 class="display-5">Users</h1>
      </div>

      <div class="row pb-5">
        <div class="col-md-6 offset-md-3 btn-toolbar justify-content-center">
          <div class="row">
            <div class="col-md-4">
              <div class="btn-group mr-2 mb-2" role="group" aria-label="First group">
                <button type="button" @click="setMode('create')" class="btn btn-secondary">Create</button>
                <button type="button" @click="setMode('search')" class="btn btn-secondary">Search</button>
              </div>
            </div>
            <div v-if="mode === 'search'" class="col-md-8">
              <div class="input-group">
                <div class="input-group-prepend">
                  <div class="input-group-text" id="btnGroupAddon">@</div>
                </div>
                <input
                  type="text"
                  class="form-control"
                  v-model="searchString"
                  placeholder="Search user by username..."
                >
              </div>
            </div>
          </div>
        </div>
      </div>

      <div class="row">
        <div
          class="accordion col-md-6 offset-md-3 mb-5"
          v-if="mode === 'search' && users.length"
          id="accordionUsers"
        >
          <div v-for="(user, id) of filteredUsers" :key="id" class="card">
            <div class="card-header pr-5 text-left" :id="'heading' + id">
              <div class="row">
                <div class="col-md-4 mt-2">
                  <span style="display: inline-block;">
                    <strong>#{{id + 1}}</strong>
                    @{{user.username}}
                  </span>
                </div>
                <div class="offset-md-4">
                  <button
                    class="btn btn-link collapsed"
                    type="button"
                    data-toggle="collapse"
                    :data-target="'#collapse' + id"
                    aria-expanded="false"
                    :aria-controls="'collapse' + id"
                  >more info...</button>
                </div>
                <button class="btn btn-default"
                  @click="deleteUser(user)">
                &times;</button>
              </div>
            </div>

            <div
              :id="'collapse' + id"
              class="collapse"
              :aria-labelledby="'heading' + id"
              data-parent="#accordionUsers"
            >
              <div class="card-body">
                <p>Real name: {{user.name}}</p>
                <p>Email: {{user.email}}</p>
                <div v-if="editUser" class="mb-2">
                  <input type="text"
                  :placeholder="user.name"
                  v-model="newUser.name">
                  <button
                    @click="updateUser(user, newUser.name)"
                    class="btn btn-outline-primary">Submit</button>
                </div>
                <button
                  @click="toggleEditUser"
                  class="btn btn-link float-right">
                  edit
                </button>
              </div>
            </div>
          </div>
        </div>

        <div v-if="mode === 'create'" class="col-md-6 offset-md-3 mb-5">
          <form>
            <div class="form-group">
              <input
                type="text"
                class="form-control"
                id="iptName"
                v-model="newUser.name"
                placeholder="Enter Name"
              >
            </div>

            <div class="form-group">
              <input
                type="text"
                class="form-control"
                id="iptName"
                v-model="newUser.username"
                placeholder="Enter Username"
              >
            </div>

            <div class="form-group">
              <input
                type="email"
                class="form-control"
                id="iptName"
                v-model="newUser.email"
                placeholder="Enter Email"
              >
            </div>
            <button
              type="submit"
              @click="createUser"
              class="btn btn-primary"
            >Submit</button>
          </form>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios'

export default {
  name: 'Index',

  data: () => ({
    searchString: '',
    users: [],
    mode: 'search',
    API: 'http://localhost:3000/users',
    newUser: {
      name: '',
      username: '',
      email: ''
    },
    editUser: false
  }),

  created () {
    axios
      .get(this.API)
      .then(response => {
        this.users = response.data
      })
      .catch(console.info)
  },

  methods: {
    setMode (newMode) {
      this.mode = newMode
    },

    createUser () {
      const userToCreate = {
        name: this.newUser.name,
        username: this.newUser.username,
        email: this.newUser.email
      }

      axios
        .post(this.API, userToCreate)
        .then(res => {
          console.info(res.data)
          this.users.unshift(res.data)
          this.newUser = {}
          this.mode = 'search'
        })
        .catch(console.error)
    },

    updateUser (user, name) {
      if (name !== '') {
        Object.assign(user, {name: name})
      }
      axios.put(`${this.API}/${user.id}`, user)
        .then(res => {
          const idx = this.users.indexOf(user)
          this.users[idx] = res.data
          this.mode = 'search'
        })
        .catch(console.error)
    },

    deleteUser (user) {
      axios.delete(`${this.API}/${user.id}`)
        .then(res => {
          const idx = this.users.indexOf(user)
          this.users.splice(idx, 1)
          this.mode = 'search'
        })
        .catch(console.error)
    },

    toggleEditUser () {
      this.editUser = !this.editUser
    }
  },

  computed: {
    filteredUsers () {
      return this.users.filter(user => {
        return user.username.indexOf(this.searchString) !== -1
      })
    }
  }
}
</script>

<style>
</style>
