<template>
  <div>
    <h4 v-if="!devices.length">{{ status }}</h4>
    <ul v-if="devices.length" class="block-list">
      <li v-for="device in devices" class="block">
        <h5>{{ device.name }}</h5>
        <ul class="prop-list">
          <li>Application: {{ device.application }}</li>
          <li>Platform: {{ device.platform }}</li>
          <li>Mobile authentication: {{ device.mobile_authentication_enabled ? "YES" : "NO" }}</li>
        </ul>
      </li>
    </ul>
  </div>
</template>

<script>
  export default {
    data() {
      return {
        devices: [],
        status: 'Loading...'
      }
    },

    created: function () {
      this.fetchDeviceList();
    },

    methods: {
      fetchDeviceList: function () {
        onegini.resource.fetch('https://demo-msp.onegini.com/resources/devices')
          .then((result) => {
            this.devices = result.json.devices;
          })
          .catch((err) => {
            console.error("Error while fetching devices: "+err.description);
            if (err.httpResponse.status == 401) {
              navigator.notification.alert('Your access token has expired, please login again', () => {
                this.$router.push('login');
              });
            } else {
              this.status = 'Could not fetch devices';
            }
          });
      }
    },
  }
</script>

<style scoped>
  h5 {
    margin-top: 0;
  }

  ul {
    list-style-type: none;
  }

  .block-list {
    padding: 0 1em;
    margin: 0 auto;
    width: 80%;
  }

  .block {
    padding: .5em;
    margin: 1.2em 0;
    background-color: rgba(0, 0, 0, .05);
    box-shadow: 0 0 5px rgba(0, 0, 0, .3);
    text-align: left;
  }

  h5 {
    margin: 0 0 .3em 0;
  }

  .prop-list {
    padding: 0;
    margin: 0;
    font-size: .8em;
  }
</style>
