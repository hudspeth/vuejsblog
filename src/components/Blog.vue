<template>
  <div class="blog-container">
    <div id="account-details">
      <div id="logged-in">
        {{userData}}
      </div>
      <div id="logged-out">
      </div>
    </div>
    <div class="login-form" v-if="showLoginForm">
      <div class="login-username">
        <label>Username</label>
        <input class="login-input" type="username" v-model="email">
      </div>
      <div class="login-password">
        <label>Password</label>
        <input class="login-input" type="password" v-model="password">
      </div>
      <button class="login-submit-btn" @click="login">Submit</button>
    </div>
    <div id="post-form" v-if="showBlogForm">
      <div class="post-title">
        <label>Enter a Title</label>
        <input class="post-input" type="text" v-model="newPost.title">
      </div>
      <div class="post-body">
        <label>Enter a Body</label>
        <textarea id="post-input" v-model="newPost.body"></textarea>
      </div>
      <button class="post-submit-btn" @click="submitBlogPost">Submit</button>
      <button id="logout-submit-btn" @click="logout">Logout</button>
    </div>
    <div class="blog-list" v-for="post in posts">
      <div class="blog-row">
        <div class="blog-title">
          {{post.title}}
        </div>
        <div class="blog-timestamp">
          {{post.timestamp.split(' ').slice(0, 5).join(' ')}}
        </div>
        <div class="blog-body" v-html="marked(post.body)">
        </div>
        <button class="post-delete-btn" v-if="showBlogForm" @click="deleteBlogPost(post)">Delete</button>
      </div>
    </div>
  </div>
</template>

<script>
import firebase from 'firebase'
import envinfo from '../../config/envinfoMINE.js' // change this to envinfo.js to use your own info

let config = {
  apiKey: envinfo.API_KEY,
  authDomain: envinfo.AUTH_DOMAIN,
  databaseURL: envinfo.DATABASE_URL,
  projectId: envinfo.PROJECT_ID,
  storageBucket: envinfo.STORAGE_BUCKET,
  messagingSenderId: envinfo.MESSAGING_SENDER_ID
}

let app = firebase.initializeApp(config)
let db = app.database()
let postsRef = db.ref('posts')

export default {
  name: 'blog',
  firebase: {
    posts: postsRef
  },
  data () {
    return {
      msg: 'My Development Blog where I attempt to be a developer that can do some stuff',
      email: '',
      password: '',
      showBlogForm: '',
      showLoginForm: '',
      userData: '',
      newPost: {
        title: '',
        body: ''
      },
      posts: []
    }
  },
  methods: {
    submitBlogPost: function () {
      // validate title and body are filled out in order to submit
      if (this.newPost.title !== '' && this.newPost.body !== '') {
        this.newPost.timestamp = Date()
        postsRef.push(this.newPost)
        console.log('submitted post successfully')
        this.newPost.title = ''
        this.newPost.body = ''
      } else {
        console.log('Title or Body are empty')
      }
    },
    deleteBlogPost: function (post) {
      console.log('POST DELETE')
      postsRef.child(post['.key']).remove()
    },
    login: function () {
      firebase.auth().signInWithEmailAndPassword(this.email, this.password)
        .then(() => {
          this.email = ''
          this.password = ''
          console.log('then do this')
          console.log(this.userData)
        })
        .catch(function (error) {
          var errorCode = error.code
          var errorMessage = error.message
          // Add form validation to say login has failed
          console.log('LOGIN FAIL')
          console.log(errorCode + ' : ' + errorMessage)
        })
    },
    logout: function () {
      firebase.auth().signOut().then(function () {
        // Sign-out successful.
        console.log('signed out')
      }).catch(function (error) {
        // An error happened.
        var errorCode = error.code
        var errorMessage = error.message
        console.log(errorCode + ' : ' + errorMessage)
      })
    }
  },
  created () {
    let initializeApp = () => {
      firebase.auth().onAuthStateChanged(user => {
        if (user) {
          // user is signed in
          var displayName = user.displayName
          var email = user.email
          user.getToken().then(accessToken => {
            this.userData = 'User: ' + displayName + ' ' + email
            this.showBlogForm = true
            this.showLoginForm = false
          })
        } else {
          // user is signed out
          this.userData = ''
          this.showBlogForm = false
          this.showLoginForm = true
        }
      }, function (error) {
        console.log(error)
      })
    }

    window.addEventListener('load', () => {
      initializeApp()
    })
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h1, h2 {
  font-weight: normal;
}

ul {
  list-style-type: none;
  padding: 0;
}

li {
  display: inline-block;
  margin: 0 10px;
}

a {
  color: #42b983;
}

.blog-list {
  text-align: left;
  margin: auto;
  width: 50%;
  padding: 10px;
}

.blog-row {
  border-top: 1px solid black;
  padding-bottom: 20px;
  position: relative;
}

.blog-title {
  display: inline;
  font-size: 30px;
}

.blog-timestamp {
  display: inline;
  position: absolute;
  right: 0px;
  font-size: 10px;
}

.blog-body {
  font-size: 15px;
}
</style>
