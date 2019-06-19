<template>
  <div>
    <h3>Hello!</h3>
    <button-lg text="App To Web Single Sign-On" @click="appToWebSingleSignOn" />
    <button-lg text="Settings" @click="$router.push('settings')" />
    <button-lg text="Logout" @click="logout" />
    <button-lg text="Deregister" @click="deregister" />
    <h3>Your devices</h3>
    <device-list />
  </div>
</template>

<script>
import ButtonLarge from '../components/Button-large.vue';
import DeviceList from '../components/Device-list.vue';

export default {
  components: {
    'button-lg': ButtonLarge,
    'device-list': DeviceList
  },

  methods: {
    appToWebSingleSignOn: function() {
      onegini.user.getAppToWebSingleSignOn("https://demo-cim.onegini.com/personal/dashboard")
          .then((result) => {
              if (cordova.platformId === 'android') {
                navigator.app.loadUrl(result.redirectUri, { openExternal: true })
              } else if (cordova.platformId === 'ios') {
                window.open(result.redirectUri, '_system');
              }
          })
          .catch((err) => {
                  navigator.notification.alert('App to web single sign on error: ' + err);
                });
    },

    logout: function() {
      onegini.user.logout()
          .then(() => {
            this.$router.push('login')
          });
    },

    deregister: function() {
      onegini.user.getAuthenticatedUserProfile()
          .then((userProfile) => {
            return onegini.user.deregister(userProfile);
          })
          .then(() => {
            this.$router.push('login');
          })
          .catch((err) => {
            navigator.notification.alert('Something went wrong! ' + err.description, () => {
              this.$router.push('login');
            });
          });
    }
  }
}
</script>
