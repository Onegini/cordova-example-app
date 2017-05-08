<template>
  <div class="view">
    <h1>Settings</h1>
    <button-lg text="Enable Mobile Authentication" @click="enrollForMobileAuthentication" />
    <button-lg text="Change PIN" @click="changePin" />
    <h3>Authenticators</h3>
    <authenticator-list />
    <button-lg text="‹ Back to Dashboard" @click="$router.push('dashboard')" />
  </div>
</template>

<script>
import ButtonLarge from '../components/Button-large.vue';
import AuthenticatorList from '../components/Authenticator-list.vue';

export default {
  components: {
    'button-lg': ButtonLarge,
    'authenticator-list': AuthenticatorList
  },

  methods: {
    enrollForMobileAuthentication: function() {
      onegini.mobileAuth.enroll()
          .then(() => {
            onegini.mobileAuth.push.enroll()
          })
          .then(() => {
            navigator.notification.alert('You are now ready to receive Mobile Authentication requests!');
          })
          .catch((err) => {
            navigator.notification.alert('Could not enroll for Mobile Authentication ' + err.description);
          });
    },

    changePin: function() {
      onegini.user.changePin()
          .onPinRequest((actions, options) => {
            let callback = (results) => {
              if (results.buttonIndex == 1) {
                actions.providePin(results.input1);
              } else {
                actions.cancel();
              }
            }

            window.plugins.pinDialog.prompt(
              `Please enter your current pin.\n${options.remainingFailureCount } out of ${options.maxFailureCount } attempts remaining`,
              callback, 'Authenticate', ['Continue','Cancel']);
          })
          .onCreatePinRequest((actions, options) => {
            let callback = (results) => {
              if (results.buttonIndex == 1) {
                actions.createPin(results.input1);
              } else {
                actions.cancel();
              }
            }

            window.plugins.pinDialog.prompt(
              `Please enter your new ${options.pinLength} digit PIN.`,
              callback, 'Authenticate', ['Continue','Cancel']);
          })
          .onSuccess(() => {
            navigator.notification.alert('PIN changed!')
          })
          .onError((err) => {
            navigator.notification.alert('Change PIN error: ' + err.description);
            if (err.code === 9003) {
              this.$router.push('login'); 
            }
          });
    }
  }
}
</script>
