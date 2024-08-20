  vue
<template>
  <div id="app" :class="weather && weather.main && getWeatherClass()">
    <main>
      <div class="search-box">
        <label for="city-input" class="search-label">Enter Your city :</label>
        <input
          id="city-input"
          type="text"
          class="search-bar"
          placeholder="Search city..."
          v-model="query"
          @keypress="fetchWeather"
        />
        <div v-if="error" class="error-message">{{ error }}</div>
      </div>
      <transition name="fade">
        <div class="weather-wrap" v-if="weather && weather.main">
          <div class="location-box">
            <div class="location animated-bounce">{{ weather.name }}, {{ weather.sys.country }}</div>
            <div class="date">{{ dateBuilder() }}</div>
          </div>
          <div class="weather-box">
            <div class="temp animated-bounce">{{ Math.round(weather.main.temp) }}°C</div>
            <div class="weather animated-bounce">{{ weather.weather[0].main }}</div>
          </div>
        </div>
      </transition>
      <div v-if="forecast && forecast.length > 0" class="forecast-wrap">
        <h2 class="forecast-title">5-Day Forecast</h2>
        <div class="forecast-box">
          <div
            class="forecast-item animated-fadeUp"
            v-for="(day, index) in forecast"
            :key="index"
          >
            <div class="forecast-date">{{ formatForecastDate(day.dt) }}</div>
            <div class="forecast-temp">{{ Math.round(day.main.temp) }}°C</div>
            <div class="forecast-weather">{{ day.weather[0].main }}</div>
          </div>
        </div>
      </div>
    </main>
  </div>
</template>

<script>
import { ref } from "vue";
export default {
  name: "App",
  setup() {
    const apiKey = "8e9fe67efd50480917691f60aa288c8a";
    const urlBase = "https://api.openweathermap.org/data/2.5/";
    const query = ref("");
    const weather = ref(null);
    const forecast = ref([]);
    const error = ref("");
    
    const fetchWeather = (e) => {
      if (e.key === "Enter") {
        fetch(`${urlBase}weather?q=${query.value}&units=metric&appid=${apiKey}`)
          .then((res) => {
            if (!res.ok) {
              throw new Error("Network response was not ok");
            }
            return res.json();
          })
          .then((data) => {
            weather.value = data;
            fetchForecast();
            query.value = ""; // Clear the input after fetching
            error.value = ""; // Clear any previous error
          })
          .catch(() => {
            error.value = "The city name is incorrect. Please try again."; // Display error message
          });
      }
    };

    const fetchForecast = () => {
      fetch(
        `${urlBase}forecast?q=${weather.value.name}&units=metric&appid=${apiKey}`
      )
        .then((res) => {
          if (!res.ok) {
            throw new Error("Network response was not ok");
          }
          return res.json();
        })
        .then((data) => {
          forecast.value = data.list.filter((item) =>
            item.dt_txt.includes("12:00:00")
          );
        })
      
    };

    const dateBuilder = () => {
      let d = new Date();
      let months = [
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
        "December",
      ];
      let days = [
        "Sunday",
        "Monday",
        "Tuesday",
        "Wednesday",
        "Thursday",
        "Friday",
        "Saturday",
      ];
      let day = days[d.getDay()];
      let date = d.getDate();
      let month = months[d.getMonth()];
      let year = d.getFullYear();
      return `${day} ${date} ${month} ${year}`;
    };

    const getWeatherClass = () => {
      if (weather.value.main.temp > 30) return "hot";
      if (weather.value.main.temp > 16) return "warm";
      if (weather.value.weather[0].main.toLowerCase().includes("rain"))
        return "rainy";
      return "cold";
    };

    const formatForecastDate = (timestamp) => {
      const date = new Date(timestamp * 1000);
      return date.toLocaleDateString("en-US", {
        weekday: "long",
        month: "short",
        day: "numeric",
      });
    };

    return {
      query,
      weather,
      forecast,
      fetchWeather,
      dateBuilder,
      getWeatherClass,
      formatForecastDate,
      error,
    };
  },
};
</script>

<style scoped>
@import url("https://fonts.googleapis.com/css2?family=Poppins:wght@300;500;700&display=swap");
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}
body {
  font-family: "Poppins", sans-serif;
  background-color: #282c34;
  color: #fff;
}
#app {
  background-size: cover;
  background-position: center;
  transition: background-image 0.5s ease-in-out;
  width: 100%;
}
#app.hot {
  background-image: url("./assets/hot.jpg");
}
#app.warm {
  background-image: url("./assets/warm.jpg");
}
#app.cold {
  background-image: url("./assets/snick.jpg");
}
#app.rainy {
  background-image: url("./assets/rainey.jpg");
}
main {
  min-height: 100vh;
  background-color: rgba(71, 78, 63, 0.692);
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}
.search-box {
  width: 100%;
  max-width: 600px;
  position: relative;
  margin-bottom: 20px;
  text-align: center;
}
.search-label {
  display: block;
  margin-bottom: 8px;
  font-size: 24px;
  color: #0f2241cf;
  padding-left: 10px;
}
.search-box .search-bar {
  width: 90%;
  padding: 15px;
  color: #fff;
  font-size: 24px;
  border: 1px solid whitesmoke;
  border-radius: 50px;
  background-color: rgba(126, 152, 187, 0.2);
  outline: none;
  transition: background-color 0.3s ease;
}
.search-box .search-bar::placeholder {
  padding: 5px;
  color: #fff;
}
.search-box .search-bar:focus {
  background-color: rgba(119, 65, 65, 0.5);
}
.error-message {
  color: #071023;
  font-size: 24px;
  position: absolute;
  top: 110%;
  left: 50%;
  transform: translateX(-50%);
  text-align: center;
}
.location-box .location {
  font-size: 36px;
  font-weight: 700;
  text-align: center;
  text-shadow: 2px 4px rgba(150, 150, 160, 0.3);
}
.location-box .date {
  font-size: 24px;
  font-weight: 300;
  text-align: center;
  margin-top: 10px;
  text-shadow: 1px 3px rgba(173, 162, 162, 0.3);
}
.weather-box {
  text-align: center;
  margin-top: 30px;
}
.weather-box .temp {
  font-size: 100px;
  font-weight: 700;
  text-shadow: 4px 8px rgba(131, 101, 101, 0.3);
  background-color: rgba(255, 255, 255, 0.2);
  border-radius: 50px;
  padding: 20px 40px;
  display: inline-block;
  margin-bottom: 10px;
}
.weather-box .weather {
  font-size: 48px;
  font-weight: 500;
  text-shadow: 3px 6px rgba(110, 105, 180, 0.3);
}
/* Forecast styling */
.forecast-wrap {
  margin-top: 40px;
  width: 100%;
  max-width: 800px;
}
.forecast-title {
  font-size: 28px;
  font-weight: 700;
  text-align: center;
  margin-bottom: 20px;
}
.forecast-box {
  display: flex;
  justify-content: space-between;
  flex-wrap: wrap;
  gap: 10px;
}
.forecast-item {
  text-align: center;
  flex: 1;
  padding: 10px;
  background-color: #ffff;
  margin: 0 10px;
  border-radius: 10px;
}
.forecast-date {
  font-size: 18px;
  font-weight: 500;
}
.forecast-temp {
  font-size: 24px;
  font-weight: 700;
  margin-top: 10px;
}
.forecast-weather {
  font-size: 20px;
  font-weight: 400;
  margin-top: 5px;
}
/* Animation styles */
@keyframes bounce {
  0%, 100% {
    transform: translateY(0);
  }
  50% {
    transform: translateY(-10px);
  }
}
.animated-bounce {
  animation: bounce 1.5s infinite;
}
/* New fade-up animation for forecast items */
@keyframes fadeUp {
  0% {
    opacity: 0;
    transform: translateY(20px);
  }
  100% {
    opacity: 1;
    transform: translateY(0);
  }
}
.animated-fadeUp {
  animation: fadeUp 1.5s ease forwards;
  opacity: 0; /* Initial opacity for smooth transition */
}
</style>
  