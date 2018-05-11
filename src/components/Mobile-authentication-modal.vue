<template>
  <div class="wrapper" v-if="request">
    <div>
      <h3>Mobile Authentication Request</h3>
      <h3>For {{request.profileId}}</h3>
    </div>
    <div class="body">
      <p v-if="fingerprintStatus">{{fingerprintStatus}}</p>
      <p v-else>{{request.message}}</p>
      <div v-if="request.type === 'push_with_pin' || fallbackToPin">
        <input type="password" pattern="[0-9]*" inputmode="numeric" v-model="pin" placeholder="Enter PIN" />
        <p>{{request.remainingFailureCount}} out of {{request.maxFailureCount}} attempts remaining</p>
      </div>
    </div>
    <div v-if="!fingerprintStatus">
      <button-lg text="✓ Accept" @click="this.accept" />
      <button-lg text="✗ Deny" @click="this.deny" />
    </div>
  </div>
</template>

<script>
import ButtonLarge from '../components/Button-large.vue';

export default {
  data () {
    return {
      pin: null,
      request: null,
      actions: null,
      fingerprintStatus: null,
      fallbackToPin: false
    }
  },

  created: function() {
    document.addEventListener('deviceready', this.listenForEvents);
  },

  methods: {
    listenForEvents: function() {
      onegini.mobileAuth.push.on('confirmation')
          .onConfirmationRequest((actions, request) => {
            this.request = request;
            this.actions = actions;
          })
          .onSuccess(() => {
            navigator.notification.alert('Authentication success!');
            this.complete();
          })
          .onError((err) => {
            navigator.notification.alert('Something went wrong! ' + err.description);
            this.complete();
          });

      onegini.mobileAuth.push.on('pin')
          .onPinRequest((actions, request) => {
            this.pin = null;
            this.actions = actions;
            this.request = request;
            this.fingerprintStatus = null;
          })
          .onSuccess(() => {
            navigator.notification.alert('PIN authentication success!');
            this.complete();
          })
          .onError((err) => {
            navigator.notification.alert('Something went wrong! ' + err.description);
            this.complete();
          });

      onegini.mobileAuth.push.on('fingerprint')
          .onFingerprintRequest((actions, request) => {
            this.actions = actions;
            this.request = request;
          })
          .onFingerprintCaptured(() => {
            this.fingerprintStatus = 'Verifying...';
          })
          .onFingerprintFailed(() => {
            this.fingerprintStatus = 'No match!';
          })
          .onSuccess(() => {
            navigator.notification.alert('Fingerprint authentication success!');
            this.complete();
          })
          .onError((err) => {
            if (err.fallbackToPin === true) {
             this.fallbackToPin = true;
             this.fingerprintStatus = null;
            } else {
              navigator.notification.alert('Something went wrong! ' + err.description);
              this.complete();
            }
          });
    },

    accept: function() {
      if (this.request.type === 'push') {
        this.actions.accept();
      } else if (this.request.type === 'push_with_pin' || this.fallbackToPin) {
        this.actions.accept({pin: this.pin});
      } else if (this.request.type === 'push_with_fingerprint') {
        this.actions.accept({
          iosPrompt: 'Authenticate as ' + this.request.profileId
        });
        this.fingerprintStatus = 'Touch sensor to start';
      }
    },

    deny: function() {
      this.actions.deny();
    },

    complete: function() {
      this.pin = null;
      this.request = null;
      this.actions = null;
      this.fingerprintStatus = null;
      this.fallbackToPin = false;
    }
  },

  components: {
    'button-lg': ButtonLarge
  }
}
</script>

<style scoped>
  .wrapper {
    display: flex;
    align-items: center;
    justify-content: space-between;
    flex-direction: column;
    box-sizing: border-box;

    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    padding: 1em;

    background-color: #fff;
    box-shadow: 0 0 1em rgba(0, 0, 0, .7);

    animation: slide .3s ease;
    animation-iteration-count: 1;
  }

  .wrapper div {
    width: 100%;
  }

  .body {
    box-sizing: border-box;
    margin: 3em 0;
    padding: 0 1em;
  }

  h3 {
    margin-bottom: 0;
  }

  h3:last-of-type {
    margin-top: .1em;
    font-weight: normal;
  }

  input {
    width: 80%;
  }

  @keyframes slide {
    from {
      top: 100%;
    }

    to {
      top: 0;
    }
  }
</style>
