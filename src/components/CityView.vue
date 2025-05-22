<template>
  <main>
    <div class="container">
      <div v-if="errorMessage" class="error-message">{{ errorMessage }}</div>
      <div v-if="!errorMessage" class="nxt">
        <h2>Next Hours</h2>
        <div v-if="forecastData" class="next-hours">
          <div v-for="(item) in forecastData.list.slice(0, 6)" :key="item.dt_txt" class="hourly-block">
            <p>{{ Math.round(item.main.temp) }} °F</p>
            <p class="precip">{{ Math.round(item.clouds.all) }}%</p>
            <img :src="`https://openweathermap.org/img/wn/${item.weather[0].icon}@2x.png`" :alt="item.weather[0].description" />
            <p>{{ formatTime(item.dt) }}</p>
          </div>
        </div>
        <div v-else>
          <p>Loading weather...</p>
        </div>
      </div>
      <div v-if="!errorMessage" class="nxt-5">
        <h2>Next 5 Days</h2>
        <div v-if="forecastData" class="next-5">
          <div v-for="(day) in getNextFiveDays()" :key="day.date" class="day-block">
            <img :src="`https://openweathermap.org/img/wn/${day.icon}@2x.png`" :alt="day.description" />
            <div class="longdate">
              <p>{{ day.displayDate }}</p>
              <p>{{ day.description }}</p>
            </div>
            <div class="hilo">
              <p>{{ Math.round(day.maxTemp) }} °F</p>
              <p>{{ Math.round(day.minTemp) }} °F</p>
            </div>
          </div>
        </div>
        <div v-else>
          <p>Loading weather...</p>
        </div>
      </div>
    <div v-if="!errorMessage" class="lastup">Last Updated on {{ new Date().toLocaleString('en-US', { month: 'short', day: 'numeric', hour: '2-digit', minute: '2-digit', hour12: false }).replace(',', '') }}</div>
    </div>
  </main>
</template>

<script>

export default {
  name: 'getCity',
  props: ['city'],
  // Pull from OpenWeatherMap API
  data () {
    return {
      api_key: '9170e0e85794088df319259526c55afd',
      url_base: 'https://api.openweathermap.org/data/2.5/forecast',
      forecastData: null,
      errorMessage: ''
    }
  },
  methods: {
    // Fetches and stores the weather data
    async fetchWeather () {
      this.errorMessage = '';
      try {
        const url = `${this.url_base}?q=${this.city.name}&appid=${this.api_key}&units=imperial`;
        const response = await fetch(url);
        const data = await response.json();

        // Error handling
        if (data.cod === "404") {
          this.forecastData = null;
          this.errorMessage = 'City not found. Try Again? 🌧️';
        } else {
          this.forecastData = data;
        }

      } catch (error) {
        console.error('Error fetching weather:', error);
      }
    },
    // Formats time and date with timezone offset
    formatTime(dt) {
      const timezoneOffset = this.forecastData.city.timezone || 0;
      const localDate = new Date((dt + timezoneOffset) * 1000);

      return new Intl.DateTimeFormat('en-US', {
        hour: 'numeric',
        minute: '2-digit',
        hour12: true,
        timeZone: 'UTC'
      }).format(localDate);
    },
    // Gets the next 5 days of weather data
    getNextFiveDays() {
      if (!this.forecastData) return [];

      const list = this.forecastData.list;
      const tzOffset = (this.forecastData.city.timezone || 0) * 1000;
      const formatter = new Intl.DateTimeFormat('en-US', {
        weekday: 'short',
        month: 'short',
        day: 'numeric'
      });
      //Loop over 24 hours
      const days = [];
      for (let i = 0; i + 7 < list.length && days.length < 5; i += 8) {
        const dayChunk = list.slice(i, i + 8);
        let min = 150, max = -150;

      // Find min and max temp
        for (const entry of dayChunk) {
          const t = entry.main.temp;
          if (t < min) min = t;
          if (t > max) max = t;
        }

        const noonEntry = dayChunk[4];
        const displayDate = formatter.format(new Date(noonEntry.dt * 1000 + tzOffset));

        days.push({
          date: noonEntry.dt_txt.split(' ')[0],
          displayDate,
          icon: noonEntry.weather[0].icon,
          description: noonEntry.weather[0].description,
          minTemp: min,
          maxTemp: max
        });
      }

      return days;
    }
  },
  watch: {
    // Watch for changes in the city prop and fetch new weather data
    city: {
      handler () {
        this.fetchWeather();
      },
      deep: true,
      immediate: true
    }
  }
};
</script>

<style>
.container {
  width: 430px;
  background: #005C8C;
  background: linear-gradient(180deg,rgba(0, 92, 140, 1) 0%, rgba(0, 157, 189, 1) 75%, rgba(205, 188, 177, 1) 75%, rgba(199, 182, 166, 1) 100%);
  padding: 10px;
}

.nxt, .nxt-5 {
  display: block;
  overflow-x: scroll;
  padding: 10px;
  background: #fdfdfd;
}

.nxt h2 {
    position: absolute;
}

.nxt h2, .nxt-5 h2 {
  font-size: 24px;
  margin-left: 10px;
  color: #333;
}

.next-hours {
  display: flex;
  justify-content: space-between;
  margin-top: 50px;
  min-width: 585px;
}

.hourly-block {
  text-align: center;
  width: 100px;
  box-shadow: 3px 2px 4px rgba(0, 0, 0, 0.1);
  padding: 10px
}

.hourly-block img{
  width: 50px;
  height: 50px;
}

.precip {
  color: #29b3d8
}

.day-block {
  display: flex;
  box-shadow: 3px 2px 4px rgba(0, 0, 0, 0.1);
}

.day-block img {
  height: 80px;
  margin-left: 22px
}

.longdate {
    display: block;
    width: 38%;
    text-align: center;
    padding: 15px 10px;
    text-transform: capitalize;
}

.hilo {
  display: flex;
  width: 34%;
  padding: 5px
}

.hilo p {
  padding: 22px 5px;
  text-align: center;
}

.lastup {
  color: white;
  padding: 2px 5px;
  background: #1565c0;
  text-align: end;
}

.error-message {
  background: white;
  padding: 100px;
  font-weight: bold;
}

</style>
