<template>
  <div id="app">
    <div class="main">
      <img src="img/logo.png"/>
      <template v-if="$route.matched.length">
        <router-view></router-view>
      </template>
      <template v-else>
        <p>{{state}}</p>
      </template>
    </div>
    <navigation-bar/>
    <mobile-authentication-modal/>
  </div>
</template>

<script>
  import MobileAuthenticationModal from '../components/Mobile-authentication-modal.vue';
  import NavigationBar from '../components/Navigation-bar.vue';

  export default {
    data() {
      return {
        state: 'Waiting for device...'
      }
    },

    created: function () {
      document.addEventListener('deviceready', this.onDeviceReady, false);
    },

    methods: {
      onDeviceReady: function() {
        const pusher = PushNotification.init({android: {},ios: {alert: "true", badge: "true", sound: "true"}});
        pusher.on('registration', (data) => {
          window.localStorage.setItem("fcmToken", data.registrationId);
        });

        pusher.on('notification', (data) => {
          if (onegini.mobileAuth.push.canHandlePushMessage(data.additionalData)) {
            onegini.mobileAuth.push.handlePushMessage(data.additionalData)
              .catch((err) => navigator.notification.alert('Push message error: ' + err.description));
          } else {
            var message = (data.title ? data.title + " " : "") + (data.message ? data.message : "");
            if (message.length > 0) {
              navigator.notification.alert(message);
            }
          }
        });

        pusher.on('error', (e) => {
          navigator.notification.alert('Push message error: ' + e.message);
        });

        this.startOnegini();
      },

      startOnegini: function () {
        this.state = 'Waiting for Onegini Plugin...';

        onegini.start()
          .then(() => {
            this.state = 'Ready!';
            return onegini.user.getAuthenticatedUserProfile();
          })
          .catch((err) => {
            if (err.code == 8005) {
              this.$router.push('login');
            } else {
              this.state = err.description;
            }
          });

      }
    },

    components: {
      'mobile-authentication-modal': MobileAuthenticationModal,
      'navigation-bar': NavigationBar
    }
  }
</script>

<style scoped>
  #app {
    font-family: 'Avenir', Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    text-align: center;
    color: #2c3e50;
  }

  .main {
    margin-top: 60px;
    margin-bottom: 80px;
  }

  img {
    display: block;
    height: 50px;
    margin: 0 auto;
  }

  h1, h2 {
    font-weight: normal;
  }
</style>
