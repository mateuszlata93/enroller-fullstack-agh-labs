<template>
  <div id="app">
    <h1>Witaj w systemie do zapisów na zajęcia</h1>

    <div v-if="authenticatedUsername">
      <UserPanel :username="authenticatedUsername" @logout="logMeOut()"></UserPanel>
      <MeetingsPage :username="authenticatedUsername"></MeetingsPage>
    </div>


    <div v-else>
      <button :class="registering ? 'button-outline' : ''" @click="registering = false" type="submit">Logowanie</button>
      <button :class="registering ? '' : 'button-outline'" @click="registering = true" type="submit">Zarejestruj się</button>

      <LoginForm v-if="!registering" @login="(user) => logMeIn(user)"></LoginForm>
      <LoginForm v-else @login="(user) => register(user)" button-label="Zaloz konto"></LoginForm>

      <h5 :v-if="message" :class="message.includes('not') ? 'alert alert-red' : ''" >{{ message }}</h5>


    </div>
  </div>
</template>

<script>
import "milligram";
import LoginForm from "./LoginForm";
import UserPanel from "./UserPanel";
import MeetingsPage from "./meetings/MeetingsPage";
import axios from "axios";

export default {
  components: { LoginForm, MeetingsPage, UserPanel},
  data() {
    return {
      authenticatedUsername: '',
      registering: false,
      userCreated: '',
      message: '',
      token: '',
    }
  },
  methods: {
    logMeIn(user) {
      axios.post('/api/tokens', user)
          .then(response => {
            this.authenticatedUsername = user.login;
            const token = response.data.token
            console.log(token)
            axios.defaults.headers.common['Authorization'] = 'Bearer ' + token;

            axios.get('/api/meetings').then(response2 => {
              console.log('here')
              console.log(response2.data.forEach(meeting => console.log(meeting)));
              MeetingsPage.meetings = [];
              MeetingsPage.meetings.push(response2.data[0]);
              console.log(MeetingsPage.meetings)
            //  MeetingsPage.methods.addNewMeeting(response2.data[0])
            } );

          })
          .catch(response => {
            this.message = 'Login not succeed'
          })
    },
    logMeOut() {
      this.authenticatedUsername = '';
      delete axios.defaults.headers.common.Authorization;
    },
    register(user) {
      axios.post('/api/participants', user)
          .then(response => {
           this.userCreated = true;
           this.message = 'User created'
          })
          .catch(response => {
            this.message = 'User not created'
            console.log('user not created')
          });
    }
  }
}
</script>

<style>
#app {
  max-width: 1000px;
  margin: 0 auto;
}

.alert {
  border: 2px green solid;
  background-color: lightblue;
  padding: 10px;
  margin: 10px;
  text-align: center;
}

.alert-red {
  font-weight: bold;
  background-color: red;
  color: black;
}
</style>
