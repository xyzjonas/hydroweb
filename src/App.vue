<template>
  <div id="app" class="mb-5">
    <nav class="navbar"
    role="navigation" aria-label="main navigation">
      <!-- logo -->
      <div class="navbar-brand">
        <a class="navbar-item" href="/">
          <img src="plant_ico.png">
        </a>
        <div class="navbar-item">
          <h5 class="title is-5">HYDRO-HUB</h5>
        </div>
      </div>
      <!-- refresh indicator -->
      <!-- <div class="navbar-end mr-2">
        <div class="level">
          <div class="level-item">
            <div class="field has-addons">
              <p v-if="interval_id" class="control">
                <button class="button is-primary is-loading" disabled>X</button>
              </p>
              <p v-else class="control">
                <button class="button is-warning" disabled>
                  <i class="fa fa-pause" aria-hidden="true"></i>
                </button>
              </p>
              <p class="control">
                <a v-on:click="toggleInterval()"
                class="button">refresh is {{ interval_id ? 'active' : 'stoped' }}
                </a>
              </p>
            </div>
          </div>
      </div>
      </div> -->
    </nav>
    <!-- DEVICES MENU NAV DROPDOWN -->
    <div v-if="devices.length > 0">
      <button id="button" v-on:click="togglePanel" class="button is-light is-fullwidth mt-2">
        <i class="fa fa-microchip"></i>
        <p class="ml-3">show all devices</p>
      </button>
    </div>
    <div class="overlay" v-if="panel_active">
        <button id="button" v-on:click="togglePanel" class="button is-light is-fullwidth">
          <i class="fa fa-microchip"></i>
          <p class="ml-3">show all devices</p>
        </button>
        <div v-if="devices.length > 0">
          <nav class="panel container">
            <a v-for="(device, d_index) in devices" :key="'d'+d_index"
              v-bind:class="{'is-active': isActive(device)}"
              v-on:click="makeActive(device)"
              class="panel-block"
              style="opacity: 1;"
            >
              <span class="panel-icon"><i v-bind:class="getFontAwesome(device)"></i></span>
              <p class="ml-2">{{ device.name }}</p>
            </a>
          </nav>
        </div>

    </div>

    <!-- DEVICE CONTAINER -->
    <div v-if="active_device != null">
      <Device :device="active_device"></Device>
    </div>

    <!-- no devices -->
    <div v-if="devices.length < 1">
      <div class="container my-5 py-5">
        <section class="hero is-large">
          <div class="hero-body">
            <p class="title">No active devices</p>
            <p class="subtitle ml-5">
              seems like you got nothing connected...
            </p>
          </div>
        </section>
      </div>
    </div>
    <div class="my-5"/>
    <div class="my-5"/>
  </div>
</template>

<script>
import axios from 'axios';
import Device from './components/Device.vue';
import Constants from './components/Constants.vue';

const INTERVAL_VALULE = 500;

export default {
  components: {
    Device,
  },

  data() {
    return {
      devices: [],
      active_tab: 0,
      active_device: null,
      server_down: false,
      panel_active: false,
      interval_id: null,
    };
  },
  methods: {
    // server comminication
    getDevices() {
      const path = `${Constants.HOST_URL}/devices`;
      axios.get(path)
        .then((res) => {
          console.debug(`${res.data.length} devices`);
          if (res.data.length < 1) {
            this.devices = [];
            return;
          }
          this.devices = res.data;
          if (!this.active_device) {
            [this.active_device] = this.devices;
            this.active_tab = this.active_device.id;
          }
        })
        .catch(() => {
          this.active_device = null;
          this.devices = [];
          this.stopInterval();
        });
    },
    getDevice(id) {
      if (!id) {
        return;
      }
      const path = `${Constants.HOST_URL}/devices/${id}`;
      console.info(`Getting: ${path}`);
      axios.get(path)
        .then((res) => {
          if (res.status !== 'success') {
            if (res.data) {
              this.active_device = res.data;
            }
          }
        })
        .catch(() => {
        });
    },

    // periodic polling
    startInterval() {
      if (this.interval_id != null) {
        // stop if running...
        console.info('stopping..');
        this.stopInterval();
      }
      this.interval_id = setInterval(() => {
        this.getDevice(this.active_device.id);
      }, INTERVAL_VALULE);
      console.error('interval started');
    },
    stopInterval() {
      clearInterval(this.interval_id);
      this.interval_id = null;
    },

    toggleInterval() {
      if (this.interval_id == null) {
        this.startInterval();
      } else {
        this.stopInterval();
      }
    },

    // active device
    isActive(dev) {
      return dev.id === this.active_tab;
    },
    makeActive(dev) {
      this.active_device = dev;
      this.active_tab = dev.id;
      this.togglePanel();
    },
    getActiveDevice() {
      return this.devices[this.active_tab];
    },

    // panel shenanigans
    showPanel() {
      this.stopInterval();
      this.getDevices();
      this.panel_active = true;
      this.disableScroll();
    },
    closePanel() {
      this.startInterval();
      this.panel_active = false;
      this.enableScroll();
    },
    disableScroll() {
      const scrollTop = window.pageYOffset || document.documentElement.scrollTop;
      const scrollLeft = window.pageXOffset || document.documentElement.scrollLeft;
      window.onscroll = function () {
        window.scrollTo(scrollLeft, scrollTop);
      };
    },
    enableScroll() {
      window.onscroll = function () {};
    },
    togglePanel() {
      if (this.panel_active) {
        this.closePanel();
      } else {
        this.showPanel();
      }
    },

    // device icon
    getFontAwesome(device) {
      if (device.type === 'serial') {
        return 'fas fa-plug';
      }
      if (device.type === 'wifi') {
        return 'fas fa-wifi';
      }
      if (device.type === 'generic') {
        return 'fa fa-microchip';
      }
      return 'fas fa-question';
    },
  },
  created() {
    this.getDevices();
    this.startInterval();
  },
};

</script>