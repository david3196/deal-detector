<template>
  <div id="app">
    <div class="wrapper">
      <nav class="navbar">
          <div class="container-fluid">
              <router-link class="navbar-brand" to="/">DealDetector</router-link>
              <div class="ml-auto" style="display: flex; justify-content: flex-end;">
                  <router-link v-if="!user" class="btn btn-outline-success" to="/login">Login</router-link>
                  <router-link v-if="!user" class="btn btn-outline-primary ms-2" to="/signup">Sign Up</router-link>
                  <router-link v-if="isAdmin" class="btn btn-outline-danger ms-2" to="/adminPage">Admin Panel</router-link>
                  <router-link v-if="user" class="btn btn-outline-success" to="/dashboard">My List</router-link>
                  <button v-if="user" @click="logout" class="btn btn-outline-red">Logout</button>    
              </div>
          </div>
      </nav>
      <router-view/>
    </div>
    <!-- <footer class="text-center">
      <p>&copy; 2023 DealDetector. All rights reserved.</p>
    </footer> -->
  </div>
</template>

<script>
import { ref, onMounted, onUnmounted, computed } from 'vue';
import { auth } from '@/firebase';
import { onAuthStateChanged } from 'firebase/auth';
import { useRouter } from 'vue-router';
import { useStore } from 'vuex';

export default {
  setup() {
    const user = ref(null);
    const router = useRouter();
    const store = useStore();
    const userObj = ref(null);

    const fetchUserData = async (email) => {
      try {
        const response = await fetch('http://localhost:3000/api/getUserByEmail?email=' + email);
        if (!response.ok) {
          throw new Error('Server responded with ' + response.status);
        }
        const data = await response.json();
        userObj.value = data;
        console.log(userObj.value.userType)
      } catch (error) {
        console.error('Failed to fetch user', error);
      }
    };
    
    const unsubscribe = onAuthStateChanged(auth, (firebaseUser) => {
      store.dispatch('setUser', firebaseUser);
      user.value = firebaseUser;
      if (firebaseUser && firebaseUser.email) {
        fetchUserData(firebaseUser.email);
      }
    });

    const isAdmin = computed(() => {
      return userObj.value && userObj.value.userType === 0;
    });

    onMounted(() => {
      // This might be redundant if the user is already set in onAuthStateChanged
      if (user.value && user.value.email) {
        fetchUserData(user.value.email);
      }
    });

    onUnmounted(() => {
      unsubscribe();
    });

    const logout = async () => {
      await auth.signOut();
      store.dispatch('setUser', null);
      user.value = null;
      userObj.value = null;
      router.push('/');
    };

    return { user, logout, userObj, isAdmin};
  },
};
</script>



<style>
html {
  box-sizing: border-box;
}

*,
*:before,
*:after {
  box-sizing: inherit;
  margin: 0;
}

body,
#app {
  margin: 0;
  padding: 0;
  display: flex;
  flex-direction: column;
  min-height: 100vh;
  font-family: Avenir, Helvetica, Arial, sans-serif;
}

#app {
  color: #1f1928;
  text-align: center;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

.wrapper {
  flex: 1;
  display: flex;
  flex-direction: column;
}

.navbar {
  padding: 10px 30px;
  background-color: #20232a;
  /* Dark background for a modern look */
  color: white;
  display: flex;
  align-items: center;
  justify-content: space-between;
}

.navbar a,
.navbar button {
  color: white;
  margin-right: 15px;
  transition: color 0.3s ease;
  border: 2px solid transparent;
  padding: 5px 10px;
  border-radius: 5px;
}

.navbar a:hover,
.navbar button:hover {
  color: #61dafb;
  /* Brighter color for hover state */
  border-color: #61dafb;
  background-color: rgba(97, 218, 251, 0.1);
}

.navbar-brand {
  font-size: 1.5em;
  font-weight: bold;
}

.nav-links {
  display: flex;
  align-items: center;
}

.footer {
  width: 100%;
  background-color: #333 !important;
  color: white;
  position: relative;
  clear: both;
  margin-top: auto;
}

router-view {
  flex: 1;
}</style>


