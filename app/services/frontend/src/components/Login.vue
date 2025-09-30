<template>
  <div id="app">
    <nav class="navbar navbar-expand-lg pt-1">
      <div class="container text-center">
        <a class="navbar-brand text-center mx-auto" href="#"><i>SPORTALIA</i></a>
      </div>
    </nav>
    <div class="container mx-auto justify-content-center mt-5">
      <div class="row mt-5 pt-5">
        <div v-if="creatingAccount" class="card mx-auto col-5 p-4">
          <h3 class="mb-4">Create an account</h3>
          <form @submit.prevent="onSubmit">
            <div class="form-label-group">
              <label for="inputEmail">Username</label>
              <input type="username" id="inputUsername" class="form-control" placeholder="Username" required autofocus
                v-model="addUserForm.username">
            </div>
            <div class="form-label-group">
              <br>
              <label for="inputPassword">Password</label>
              <input type="password" id="inputPassword" class="form-control" placeholder="Password" required
                v-model="addUserForm.password">
            </div>
            <div class="text-center">
              <button type="submit" class="btn btn-primary w-100 d-block mt-4">Submit</button>
              <button class="btn btn-outline-success w-100 d-block mt-3" @click="backToLogin">Back To Log In</button>
            </div>
          </form>
        </div>
        <!-- SIGN IN -->
        <div v-else class="card mx-auto col-5 p-4">
            <h3 class="mb-4">Sign In</h3>
            <div class="form-label-group">
              <label for="inputEmail">Username</label>
              <input type="username" id="inputUsername" class="form-control"
              placeholder="Username" required autofocus v-model="username">
            </div>
            <div class="form-label-group">
              <br>
              <label for="inputPassword">Password</label>
              <input type="password" id="inputPassword" class="form-control"
              placeholder="Password" required v-model="password">
            </div>
            <div class="text-center">
              <button class="btn btn-primary w-100 d-block mt-4" @click="checkLogin">Sign In</button>
              <button class="btn btn-success w-100 d-block mt-3" @click="initCreateForm">Create Account</button>
              <button class="btn btn-outline-success w-100 d-block mt-3" @click="backToMatches">Back to Matches</button>
            </div>
        </div>
      </div>
     </div>
  </div>
</template>

<script>
import axios from 'axios'

// Create an axios instance with a reasonable timeout to better surface network errors
const http = axios.create({
  timeout: 10000 // 10s
})

// Derive API base from the page hostname to handle localhost/127.0.0.1 seamlessly
const API_BASE = `${window.location.protocol}//${window.location.hostname}:8000`

export default {
  name: 'Login',
  data () {
    return {
      logged: false,
      username: null,
      password: null,
      token: null,
      creatingAccount: false,
      addUserForm: {
        username: null,
        password: null
      }
    }
  },
  methods: {
    checkLogin () {
      const parameters = 'username=' + this.username + '&password=' + this.password
  const path = `${API_BASE}/login`
      const config = {
        headers: { 'Content-Type': 'application/x-www-form-urlencoded' }
      }
      http.post(path, parameters, config)
        .then((res) => {
          this.logged = true
          this.token = res.data.access_token
          this.$router.push({ path: '/', query: { username: this.username, logged: this.logged, token: this.token } })
        })
        .catch((error) => {
          // eslint-disable-next-line
          alert('Username: ' + parameters.username + ' Password: ' + parameters.password)
          console.error(error)
          const isNetwork = !error.response
          alert(isNetwork ? 'Network error. Is the backend up on http://localhost:8000?' : 'Username or Password incorrect')
          // this.backToLogin()
        })
    },
    initCreateForm () {
      this.creatingAccount = true
      this.addUserForm.username = null
      this.addUserForm.password = null
    },
    onSubmit () {
      // basic client-side validation to avoid 422 surprises
      if (!this.addUserForm?.username) {
        alert('Please enter a username')
        return
      }
      if (!this.addUserForm?.password || this.addUserForm.password.length < 8) {
        alert('Password must be at least 8 characters long')
        return
      }

      const parameters = {
        username: this.addUserForm.username,
        password: this.addUserForm.password
      }
  const path = `${API_BASE}/account`
      http.post(path, parameters)
        .then(() => {
          alert('Account created')
          this.backToLogin()
        })
        .catch((error) => {
          // Try to surface the actual backend error
          let msg = 'Failed to create account'
          const resp = error && error.response
          if (resp) {
            if (resp.data?.detail) {
              msg = typeof resp.data.detail === 'string' ? resp.data.detail : JSON.stringify(resp.data.detail)
            } else if (Array.isArray(resp.data)) {
              // FastAPI 422 validation errors come as a list
              msg = resp.data.map(e => e?.msg || JSON.stringify(e)).join(', ')
            } else if (typeof resp.data === 'string') {
              msg = resp.data
            }
          } else if (error?.message) {
            msg = 'Network error. Is the backend up on http://localhost:8000?\n' + error.message
          }
          alert(msg)
          console.error(error)
        })
    },
    backToLogin () {
      this.creatingAccount = false
      this.username = null
      this.password = null
    },
    backToMatches () {
      this.$router.push({ path: '/' })
    },
    created () {
      this.backToLogin()
    }
  }
}
</script>

<style scoped>
#app {
  background-image: url('../assets/background.jpg');
  height: 100vh;
}
  .navbar {
    background-color: #000000;
    margin: 0;
    width: 100%;
  }
  .navbar-brand {
    color: #00ff15;
    font-size: 1.4rem;
  }
  .card {
    background-color: #000000c6;
    border: none;
    color: white;
    border-radius: 10px;
  }
  .form-label-group {
    margin-bottom: 1em;
  }
  .form-label-group > input {
    border: 1px solid #00ff15;
  }
</style>

