<template>
  <q-page class="flex column" :class="bgClass">
    <div class="col q-pt-lg q-px-md">
      <q-input
        @keyup.enter="getWeatherBySearch"
        v-model="search"
        placeholder="Search"
        dark
        borderless
      >
        <template v-slot:before>
          <q-btn @click="getLocation" round dense flat icon="my_location" />
        </template>

        <template v-slot:append>
          <q-btn @click="getWeatherBySearch" round dense flat icon="search" />
        </template>
      </q-input>
    </div>

    <template v-if="weatherData">
      <div class="col text-white text-center">
        <div class="text-h4 text-weight-light">
          {{ weatherData.name }} ({{ weatherData.sys.country }})
        </div>
        <div class="text-h6 text-weight-light">
          {{ weatherData.weather[0].main }}
        </div>
        <div class="text-h1 text-weight-thin q-my-lg">
          {{ Math.round(weatherData.main.temp) }}&deg;C
        </div>
      </div>

      <div class="col text-center">
        <img
          :src="`http://openweathermap.org/img/wn/${weatherData.weather[0].icon}@2x.png`"
        />
      </div>
    </template>

    <template v-else>
      <div class="col column text-center text-white">
        <div class="col text-h2 text-weight-thin">Quasar<br />Weather</div>

        <q-btn @click="getLocation" class="col" flat>
          <q-icon left size="3em" name="my_location" />
          <div>Find My Location</div>
        </q-btn>
      </div>
    </template>

    <div class="col skyline"></div>
  </q-page>
</template>

<script>
export default {
  name: "PageIndex",
  data() {
    return {
      search: "",
      weatherData: null,
      apiUrl: "https://api.openweathermap.org/data/2.5/weather",
      apiKey: "11bbfba58b96a7ccb3864bc1e59bbd96",
      lat: "",
      lon: "",
    };
  },
  methods: {
    getLocation() {
      this.$q.loading.show();

      if (this.$q.platform.is.electron) {
        this.$axios.get("https://freegeoip.app/json/").then((response) => {
          this.lat = response.data.latitude;
          this.lon = response.data.longitude;
          this.getWeatherByCoords();
        });
      } else {
        navigator.geolocation.getCurrentPosition((position) => {
          this.lat = position.coords.latitude;
          this.lon = position.coords.longitude;

          this.getWeatherByCoords();
        });
      }
    },
    getWeatherByCoords() {
      this.$axios(
        `${this.apiUrl}?lat=${this.lat}&lon=${this.lon}&appid=${this.apiKey}&units=metric`
      )
        .then((response) => {
          this.weatherData = response.data;

          this.$q.loading.hide();
        })
        .catch(() => {
          alert("Problem with location!");

          this.$q.loading.hide();
        });
    },
    getWeatherBySearch() {
      this.$q.loading.show();

      this.$axios(
        `${this.apiUrl}?q=${this.search}&appid=${this.apiKey}&units=metric`
      )
        .then((response) => {
          this.weatherData = response.data;

          this.$q.loading.hide();
        })
        .catch(() => {
          alert("Please enter the correct city name!");

          this.$q.loading.hide();
        });
    },
  },
  computed: {
    bgClass() {
      if (this.weatherData) {
        if (this.weatherData.weather[0].icon.endsWith("n")) {
          return "bg-night";
        } else {
          return "bg-day";
        }
      }
    },
  },
};
</script>

<style lang="sass">
.q-page
  background: linear-gradient(to bottom, #136a8a, #267871)
  &.bg-night
    background: linear-gradient(to bottom, #232527, #414345)
  &.bg-day
    background: linear-gradient(to bottom, #00b4db, #0083b0)
.skyline
  flex: 0 0 100px
  background: url("../statics/skyline.png")
  background-size: contain
  background-position: center bottom
</style>
