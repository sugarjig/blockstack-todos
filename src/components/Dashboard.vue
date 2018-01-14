<template>
  <div class="hello">
    <div class="container">
      <div class="row">
        <div class="col-md-8 col-md-offset-2">
          <h1 class="page-header">BlockPass App
            <img :src="user.avatarUrl() ? user.avatarUrl() : '/avatar-placeholder.png'" class="avatar">
            <small><span class="sign-out">(<a href="#" @click.prevent="signOut">Sign Out</a>)</span></small>
          </h1>
          <h2 class="user-info">
            <small>
              {{ user.name() ? user.name() : 'Nameless Person'}}'s Passwords
            </small>
            <small class="pull-right">
            {{ user.username ? user.username : user.identityAddress }}
            </small>

          </h2>
          <form @submit.prevent="addPassword" :disabled="! username || ! password">
            <div class="input-group">
              <input v-model="username" type="text" class="form-control" placeholder="Enter a username..." autofocus>
              <input v-model="password" type="text" class="form-control" placeholder="Enter a password..." autofocus>              
              <span class="input-group-btn">
                <button class="btn btn-default" type="submit" :disabled="! username || ! password">Add</button>
              </span>
            </div>
          </form>

          <ul class="list-group">
            <li v-for="password in passwords"
              class="list-group-item"
              :class="{completed: password.completed}"
              :key="password.id">
              {{password.username}}: {{password.text}}
            </li>
          </ul>

        </div>
      </div>
    </div>
  </div>
</template>

<script>
var STORAGE_FILE = 'passwords.json'

export default {
  name: 'dashboard',
  props: ['user'],
  data () {
    return {
      blockstack: window.blockstack,
      passwords: [],
      username: '',
      password: '',
      uidCount: 0
    }
  },
  watch: {
    passwords: {
      handler: function (passwords) {
        const blockstack = this.blockstack
        const encrypt = true
        return blockstack.putFile(STORAGE_FILE, JSON.stringify(passwords), encrypt)
      },
      deep: true
    }
  },
  mounted () {
    this.fetchData()
  },
  methods: {
    addPassword () {
      if (!this.password.trim()) {
        return
      }
      this.passwords.unshift({
        id: this.uidCount++,
        username: this.username.trim(),
        text: this.password.trim(),
        completed: false
      })
      this.username = ''
      this.password = ''
    },

    fetchData () {
      const blockstack = this.blockstack
      const decrypt = true
      blockstack.getFile(STORAGE_FILE, decrypt)
      .then((passwordsText) => {
        var passwords = JSON.parse(passwordsText || '[]')
        passwords.forEach(function (password, index) {
          password.id = index
        })
        this.uidCount = passwords.length
        this.passwords = passwords
      })
    },

    signOut () {
      this.blockstack.signUserOut(window.location.href)
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="scss" scoped>

input::placeholder {
  color: grey;
}

label {
  margin-bottom: 0;
  // width: 100%;
  cursor: pointer;
  input[type="checkbox"] {
    margin-right: 5px;
  }
}
.list-group-item {
  &.completed label {
    text-decoration: line-through;
  }

  .delete {
    display: none;
  }

  &:hover .delete {
    display: inline;
    color: grey;
    &:hover {
      text-decoration: none;
      color: red;
    }
  }
}
</style>
