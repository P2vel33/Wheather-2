<template>
  <div>
    <div>
      <div v-if="errorMessage">
        <p>
          {{ errorMessage }}
        </p>
      </div>
      <template v-else>
        <div v-if="loading">Loading</div>
        <div v-else>
          <form @submit.prevent>
            <input v-model="city" type="text" placeholder="Введите название города" >
          </form>
          <button @click="successCity(city)">Узнать</button>
          <h5>
            <span id="city">Погода в {{ city }}</span>
          </h5>
          <div>
            <span id="value">{{ weatherData.value }}</span>
            <span>°</span>
          </div>
          <ul role="list">
            <li>
              <span>
                Ощущается как
                <span id="feels_like">{{ weatherData.feelsLike }}</span>
              </span>
            </li>
            <li>
              <span>Влажность 
                <span id="humidity">{{ this.weatherData.humidity }}
                </span>%
              </span>
            </li>
            <li>
              <span>
                Атмосферное давление
                <span id="pressure">{{ this.weatherData.pressure }}</span> мм
                рт. ст.
              </span>
            </li>
          </ul>
        </div>
      </template>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      // timer: setTimeout(this.getWheatherByCity(city),1000),
      city: "",
      errorMessage: "",
      loading: true,
      weatherData: {},
      API_KEY : import.meta.env.VITE_APP_API_KEY,
    }
  },


  
  computed: {
    name(){
      return this.data != null ? this.data.weatherData.city : '';
    },
    
    value(){
      return this.data != null ? this.data.weatherData.value + ' °': '';
    },
    
    feelsLike(){
      return this.data != null ? this.data.weatherData.feelsLike : '';
    },

    humidity(){
      return this.data != null ? this.data.weatherData.humidity + ' %' : '';
    },

    pressure(){
      return this.data != null ? this.data.weatherData.pressure + 'мм рт. ст.' : '';
    }
  },


  
  watch: {
    city(value){
      value != '' ? this.timer = null : console.log('ad');
    }
  },


  
  methods: {

    async getWheatherByCity(city){
      try {
        console.log(city);
        const response = await fetch(
          `https://api.openweathermap.org/data/2.5/weather?q=${city}&appid=${this.API_KEY}&lang=ru`
        );
        // console.log(response.json());
        return await response.json();
      } catch (e) {
        this.errorMessage = "Can't load weather data from the remote host";
      }
    },

    async getWeatherByCoords(lat, lon) {
      try {
        const response = await fetch(
          `https://api.openweathermap.org/data/2.5/weather?lat=${lat}&lon=${lon}&appid=${this.API_KEY}&lang=ru`
        );
        return await response.json();
      } catch (e) {
          this.errorMessage = "Can't load weather data ";
      }
    },
    
    async successCity(city){
      const weather = await this.getWheatherByCity(city);
      if (!weather) {
        this.errorMessage = "Can't load weather data from ";
        this.loading = false;
      } else {
        this.loading = false;
        this.setWeatherData(weather);
      }
    },

    async success(pos) {
      const { latitude, longitude } = pos.coords;
      const weather = await this.getWeatherByCoords(latitude, longitude);
      if (!weather) {
        this.errorMessage = "Can't load weather data from ";
        this.loading = false;
      } else {
        this.loading = false;
        this.setWeatherData(weather);
      }
    },
    
    async error(err) {
      if (err.code === 1) {
        console.log("Not enough permissions");
      } else if (err.code === 2) {
        console.log("Position unavailable");
      } else if (err.code === 3) {
        console.log("Timeout exceeded");
      } else {
        console.log("Unknown error");
      }
      const { latitude, longitude } = await this.getLocationByIp();
      this.weatherData = await this.getWeatherByCoords(latitude, longitude);
      if (!this.weatherData) {
        throw new Error("Can't load weather data from the ");
      } else {
        setWeatherData(this.weatherData);
      }
    },

    async getLocationByIp() {
      try {
        const response = await fetch("https://ipapi.co/json");
        const { latitude, longitude } = await response.json();
        return { latitude, longitude };
      } catch (e) {
        this.errorMessage = "Can't load weather data from the remote ";
      }
    },

    setWeatherData(data) {
      this.weatherData = {
        city: data?.name,
        value: Number(data?.main?.temp - 273.15).toFixed(1),
        feelsLike: Number(data?.main?.feels_like - 273.15).toFixed(1),
        humidity: Number(data?.main?.humidity).toFixed(1),
        pressure: Number(data?.main?.pressure * 0.750062).toFixed(1),
      };
    },
  },
  
  mounted() {
    navigator.geolocation.getCurrentPosition(this.success, this.error, {
      enableHighAccuracy: true,
      timeout: 5000,
      maximumAge: 0,
    });
  },
};
</script>

<style scoped>

</style>
