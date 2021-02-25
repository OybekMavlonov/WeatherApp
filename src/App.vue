<template>
  <div id="app" :class="typeof weather.main != 'undefined'">
    <v-app id="inspire">
      <v-container class>
        <!-- <div class="search-box">
          <input
            type="text"
            class="search-bar"
            placeholder="Search..."
            v-model="query"
            @keyup.enter="fetchWeatherAxios"
          />
        </div>-->

        <v-text-field
          v-model="query"
          filled
          shaped
          clearable
          background-color="#fff"
          color="#313131"
          placeholder="Search..."
          @keyup.enter="fetchWeatherAxios()"
        ></v-text-field>

        <v-row justify="space-between" v-if="typeof weather.city != 'undefined'">
          <v-col md="6">
            <v-card class="pa-4" outlined tile>
              <h2 class="city-name">{{ weather.city.name }}, {{ weather.city.country }}</h2>
              <strong
                class="current-day"
              >Weather Condition in: {{ dateBuilder(weather.list[0].dt) }}</strong>

              <div class="current-forecast">
                <span>
                  <img
                    :src="
                      'http://openweathermap.org/img/w/' +
                        weather.list[0].weather[0].icon +
                        '.png'
                    "
                  />
                </span>
                <span>
                  <strong>
                    <span
                      v-if="Math.round(weather.list[0].main.temp) > 0"
                      style="
    margin-bottom: 5px;
    font-size: 35px;
"
                    >+</span>
                    {{ Math.round(weather.list[0].main.temp) }}&deg;
                  </strong>
                </span>
              </div>

              <div class="current-forecast-desc">{{ weather.list[0].weather[0].main }}</div>

              <div class="current-forecast-details">
                <div class="column-1">
                  <p>Humidity: {{ weather.list[0].main.humidity }}%</p>
                  <p>
                    Wind: {{ windDirection() }},
                    {{ Math.round(weather.list[0].wind.speed) }} m/s
                  </p>
                  <p>
                    Pressure: {{ weather.list[0].main.grnd_level }} hPa (on the
                    ground level)
                  </p>
                </div>
                <div class="column-2">
                  <p>precipitation: {{Math.round(weather.list[0].pop*100) }} %</p>
                  <p>Sunrise: {{ calcTime(weather.city.sunrise, weather.city.timezone) }}</p>
                  <p>Sunset: {{ calcTime(weather.city.sunset, weather.city.timezone) }}</p>
                </div>
              </div>
            </v-card>
          </v-col>

          <v-col md="6">
            <v-card class="pa-4" outlined tile>
              <div class="table-heade">
                <h4 class="text-center">Weather information for 4 days</h4>
              </div>
              <v-simple-table dense>
                <template v-slot:default>
                  <thead>
                    <tr>
                      <th class="text-left">Day</th>
                      <th class="text-center">Temperature</th>
                      <th class="text-center">Description</th>
                      <th class="text-center">Probability of precipitation</th>
                    </tr>
                  </thead>
                  <tbody>
                    <tr v-for="(item, index) in weather.list" :key="index">
                      <td v-if="index % 8 == 0 && index != 0">
                        <strong v-if="formattedDate(item.dt).includes(tomorrow())">Tomorrow</strong>

                        <strong v-else>{{ formattedDay(item.dt) }}</strong>
                        <div>{{ formattedDate(item.dt) }}</div>
                      </td>
                      <td
                        class="text-center"
                        v-if="index % 8 == 0 && index != 0"
                        style="
    font-size: 20px;
"
                      >
                        <img
                          style="
    margin-right: 5px;"
                          :src="
                      'http://openweathermap.org/img/w/' +
                        item.weather[0].icon +
                        '.png'
                    "
                        />
                        <span v-if="Math.round(item.main.temp) > 0">+</span>
                        {{ Math.round(item.main.temp) }} &deg;
                      </td>
                      <td
                        class="text-center"
                        v-if="index % 8 == 0 && index != 0"
                      >{{ item.weather[0].main }}</td>
                      <td
                        class="text-center"
                        v-if="index % 8 == 0 && index != 0"
                      >{{ Math.round(item.pop*100) }} %</td>
                    </tr>
                  </tbody>
                </template>
              </v-simple-table>
            </v-card>
          </v-col>
        </v-row>
      </v-container>
    </v-app>
  </div>
</template>

<script>
var d2d = require("degrees-to-direction");
import axios from "axios";

export default {
  name: "app",
  data() {
    return {
      api_key: "cdd3915d3989e0428cec76a6bec440b4",
      url_base: "https://api.openweathermap.org/data/2.5/",
      query: "",
      weather: {},
      months: [
        "January",
        "February",
        "March",
        "April",
        "May",
        "June",
        "July",
        "August",
        "September",
        "October",
        "November",
        "December"
      ],
      days: [
        "Sunday",
        "Monday",
        "Tuesday",
        "Wednesday",
        "Thursday",
        "Friday",
        "Saturday"
      ]
    };
  },
  methods: {
    fetchWeather() {
      fetch(
        `${this.url_base}forecast?q=${this.query}&units=metric&appid=${this.api_key}`
      )
        .then(res => {
          return res.json();
        })
        .then(data => {
          this.weather = data;
          const today = data.list[0];
          console.log(
            `Temperature in ${data.city.name} stay between ${today.main.temp_min} and ${today.main.temp_max}. Humidity: ${today.main.humidity}`
          );
        })
        .catch(err => console.log(err));
    },
    async fetchWeatherAsync() {
      const result = await fetch(
        `${this.url_base}forecast?q=${this.query}&units=metric&appid=${this.api_key}`
      );
      const data = await result.json();

      this.weather = data;
    },
    fetchWeatherAxios() {
      axios
        .get(
          `${this.url_base}forecast?q=${this.query}&units=metric&appid=${this.api_key}`
        )
        .then(res => {
          const data = res.data;
          this.weather = data;
        })
        .catch(err => console.log(err));
      this.query = "";
    },
    formattedDate(time) {
      let unix_timestamp = time;
      let date = new Date(unix_timestamp * 1000);
      let hours = date.getHours();
      let minutes = "0" + date.getMinutes();
      let exact_date = date.getDate();
      let month = this.months[date.getMonth()];
      let formattedTime = hours + ":" + minutes.substr(-2);

      return `${exact_date} ${month}, ${formattedTime}`;
    },
    dateBuilder(time) {
      let unix_timestamp = time;
      let date = new Date(unix_timestamp * 1000);
      let hours = date.getHours();
      let minutes = "0" + date.getMinutes();
      let exact_date = date.getDate();
      let month = this.months[date.getMonth()];
      let formattedTime = hours + ":" + minutes.substr(-2);

      let day = this.days[date.getDay()];
      let year = date.getFullYear();

      return `${day}, ${exact_date} ${month}, ${formattedTime}, ${year}`;
    },
    calcTime(time, timezone) {
      let unix_timestamp = time;
      // Create a new JavaScript Date object based on the timestamp
      // multiplied by 1000 so that the argument is in milliseconds, not seconds.
      var d = new Date(unix_timestamp * 1000);

      // convert to msec
      // add local time zone offset
      // get UTC time in msec
      var utc = d.getTime() + d.getTimezoneOffset() * 60000;

      // create new Date object for different city
      // using supplied offset
      var offset = timezone / 3600;
      var nd = new Date(utc + 3600000 * offset);

      // Hours part from the timestamp
      var hours = nd.getHours();
      // Minutes part from the timestamp
      var minutes = "0" + nd.getMinutes();

      // Will display time in 10:30:23 format
      var formattedTime = hours + ":" + minutes.substr(-2);

      return formattedTime;
    },
    tomorrow() {
      let d = new Date();
      let date = d.getDate() + 1;
      let month = this.months[d.getMonth()];
      return `${date} ${month}`;
    },
    formattedDay(day) {
      let unix_timestamp = day;

      var date = new Date(unix_timestamp * 1000);

      let dayOfWeek = this.days[date.getDay()];

      return `${dayOfWeek}`;
    },
    windDirection() {
      return d2d(this.weather.list[0].wind.deg);
    }
  }
};
</script>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}
body {
  font-family: "montserrat", sans-serif;
}
.theme--light.v-application {
  background: #eee !important;
}
.v-text-field input {
  font-size: 18px;
}

.v-input__icon--clear .v-icon--disabled {
    visibility: visible;
}
.mdi-close::before {
  content: "\F0156";
}
.mdi:before,
.mdi-set {
  display: inline-block;
  font: normal normal normal 24px/1 "Material Design Icons";
  font-size: inherit;
  text-rendering: auto;
  line-height: inherit;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}
.search-box {
  width: 100%;
  margin-bottom: 30px;
}
.search-box .search-bar {
  display: block;
  width: 100%;
  padding: 15px;
  color: #313131;
  font-size: 20px;
  appearance: none;
  border: none;
  outline: none;
  background: none;
  box-shadow: 0px 0px 8px rgba(0, 0, 0, 0.25);
  background: white;
  border-radius: 0px 16px 0px 16px;
  transition: 0.4s;
}
.search-box .search-bar:focus {
  box-shadow: 0px 0px 16px rgba(0, 0, 0, 0.25);
  background-color: rgba(255, 255, 255, 0.75);
  border-radius: 16px 0px 16px 0px;
}
.city-name {
  font-size: 21px;
  font-weight: bold;
  margin: 0 0 8px 0;
}
.current-day {
  font-size: 15px;
  margin: 10px 0 0 0;
}
.current-forecast {
  height: 64px;
  font-size: 28px;
  color: #666;
  margin: 10px 0 0 0;
}
.current-forecast span {
  display: inline-flex;
  vertical-align: middle;
}
.current-forecast strong {
  font-size: 42px;
  font-weight: normal;
  color: #222;
  margin-left: 10px;
  margin-right: 10px;
}
.current-forecast-desc {
  font-size: 18px;
  margin: 10px 0 10px 0;
  border-bottom: 1px solid #ccc;
  padding-bottom: 10px;
}

.current-forecast-details {
  display: flex;
  justify-content: space-between;
  color: #666;
  margin: 5px;
}

table tr td {
  padding: 4px 10px !important;
  font-size: 20px;
}
</style>
