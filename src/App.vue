<template>
  <div id="app">
    <header>
      <nav>
        <router-link class="logo" to="/"> Cal Track </router-link>
        <ul>
          <li v-if="!isLoggedIn"><router-link to="/sign-in"> Login </router-link></li>
          <li v-if="!isLoggedIn"><router-link to="/register"> Sign Up </router-link></li>
          <li @click="logout" v-if="isLoggedIn"><router-link  to="/"> Logout </router-link></li>
          <li v-if="isLoggedIn"><router-link  to="/profile"> Account </router-link></li>
        </ul>
      </nav>
    </header>
    <router-view/>
  </div>
</template>

<script>
import firebase from 'firebase';
  export default{
    data() {
      return {
        isLoggedIn: false
      }
    },
    created() {
    firebase.auth().onAuthStateChanged((user) => {
      if (user) {
        // User is logged in
        this.isLoggedIn = true;
      } else {
        // User is logged out
        this.isLoggedIn = false;
      }
    });
  },
  methods: {
    logout() {
      firebase.auth().signOut()
    }
  },
  
  }

</script>

<style>

*{
  margin:0;
}

header {
  background-color: #fff;
  box-shadow: 0 2px 4px rgba(0,0,0,.1);
}

nav {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 20px;
  background-color: #4caf50;
  margin-bottom: 3em;
}

.logo {
  font-size: 24px;
  font-weight: bold;
  color: black;
}

ul {
  list-style-type: none;
  padding: 0;
  margin: 0;
}

li {
  background-color: #fff;
  padding: 10px;
  border-radius: 5px;
  display: inline-block;
  margin: 0 10px;
}

a {
  color: black;
  text-decoration: none;
  font-weight: bold;
  border-bottom: 2px solid transparent;
  transition: border-bottom .2s;
}

a:hover {
  border-bottom: 2px solid #42b983;
}


#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}

/* nav {
  padding: 30px;
} */

/* nav a {
  font-weight: bold;
  color: #2c3e50;
}

nav a.router-link-exact-active {
  color: #42b983;
} */
</style>
