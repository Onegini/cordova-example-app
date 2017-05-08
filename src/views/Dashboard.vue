<template>
  <div>
    <h3>Hello!</h3>
    <button-lg text="Settings" @click="$router.push('settings')" />
    <button-lg text="QR login" @click="qrLogin" />
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
    },

    qrLogin: function() {
      QRScanner.scan(function (err, data) {
        QRScanner.hide();
        if (err) {
          // an error occurred, or the scan was canceled (error code `6`)
        }
        else {

          onegini.mobileAuth.otp.handleRequest({
            otp : data
          })
              .onConfirmationRequest((actions, request) => {
                console.log("New OTP mobile authentication request", request);
                actions.accept();
              })
              .onSuccess(() => {
                alert("OTP Mobile authentication request success!");
              })
              .onError((err) => {
                alert("OTP Mobile authentication request failed!\n\n" + err.description);
              });
        }
      });

      QRScanner.show();
    }
  }
}
</script>
