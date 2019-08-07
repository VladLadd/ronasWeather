<template>
  <div id="app">
    <div class="d-flex justify-content-between align-items-center flex-column">
      <div class="row justify-content-between w-100 mx-0">
        <div class="col-5 pl-0">
          <div class="d-flex" v-if="findInput">
            <input v-model="nameTown" class="form-control form-control-sm py-3" type="text" placeholder="Введите название города"><button class="btn_ok" @click="changeTown">OK</button>
          </div>
          <div class="row mx-0" v-else>
            <div class="d-flex align-items-start flex-column">
              <h1>{{dataWeather.name}}</h1>
              <button class="find_input" type="button" @click="showFind">Сменить город</button>
            </div>
            <div class="d-flex align-items-end ml-3">
              <button class="find_input" @click="geoUser"><i class="fas fa-location-arrow pr-1"></i>Мое местоположение</button>
            </div>
          </div>
        </div>
        <div class="col-5 d-flex justify-content-end align-items-center pr-0">
          <span>C°</span>
          <div class="custom-control custom-switch">
            <input type="checkbox" class="custom-control-input" id="customSwitch1" @change="changeUnits">
            <label class="custom-control-label" for="customSwitch1"></label>
          </div>
          <span>F°</span>
        </div>
      </div>
    </div>

    <div class="d-flex justify-content-center align-items-center flex-column h-50">
      <div class="main_weather">
        <h1>{{dataWeather.temp}}°</h1>
        <div>
          <img src="./assets/cloud.png" v-if="dataWeather.cloudsType === 'Clouds'">
          <img src="./assets/sun.png" v-else-if="dataWeather.cloudsType === 'Clear'">
          <img src="./assets/rain.png" v-else-if="dataWeather.cloudsType === 'Rain'">
          <img src="./assets/strom.png" v-else-if="dataWeather.cloudsType === 'Storm'">
          <img src="./assets/partly_cloudy.png" v-else>
          <span>{{dataWeather.clouds}}</span>
        </div>
      </div>
    </div>

    <div class="row justify-content-between align-items-center mx-0">
      <div class="d-flex flex-column align-items-start">
        <span>Ветер</span>
        <h2 v-if="units === 'metric'">{{dataWeather.wind.speed}} м/с, {{dataWeather.windDegName}}</h2>
        <h2 v-else>{{dataWeather.wind.speed}} фут/с, {{dataWeather.windDegName}}</h2>
      </div>
      <div class="d-flex flex-column align-items-start">
        <span>Давление</span>
        <h2>{{dataWeather.pressure}} мм рт. ст.</h2>
      </div>
      <div class="d-flex flex-column align-items-start">
        <span>Влажность</span>
        <h2>{{dataWeather.humidity}}%</h2>
      </div>
      <div class="d-flex flex-column align-items-start">
        <span>Вероятность дождя</span>
        <h2>{{dataWeather.cloudsAll}}%</h2>
      </div>

    </div>
  </div>
</template>


<script>
import axios from 'axios';

export default {
  name: 'app',
  components: {
  },
  data() {
    return {
      findInput: false,
      nameTown: 'Omsk',
      dataWeather: {
          temp:'',
          logo: null,
          clouds:'',
        cloudsType:'',
          wind: {
            speed:'',
            deg: null,
          },
          windDegName:'',
          pressure: '',
          humidity:'',
          cloudsAll:'',
      },
      units: 'metric'
    }
  },
  methods: {
    showFind() {
      this.findInput = true;
    },
     getData() {
        axios.get(`http://api.openweathermap.org/data/2.5/weather?q=` + this.nameTown + `&appid=e275f27c1a9f13ff0b3f1746b12fd26c&units=` + this.units + `&lang=ru`).then(res => {
          this.dataWeather.temp = Math.trunc(res.data.main.temp);
          this.dataWeather.name = res.data.name;
          this.dataWeather.clouds = res.data.weather[0].description;
          this.dataWeather.cloudsType = res.data.weather[0].main;
          this.dataWeather.wind.speed = res.data.wind.speed;
          this.dataWeather.wind.deg = res.data.wind.deg;
          this.dataWeather.pressure = res.data.main.pressure;
          this.dataWeather.humidity = res.data.main.humidity;
          this.dataWeather.cloudsAll = res.data.clouds.all;

          this.logoWeather();
          this.degWind();
        });
    },
    changeTown() {
      this.findInput = false;
      this.getData();
    },
    degWind() {
      let deg = this.dataWeather.wind.deg;
      if (deg === 360){
        this.dataWeather.windDegName = 'Южный'
      } else if (deg === 270){
        this.dataWeather.windDegName = 'Восточный'
      } else if (deg === 180){
        this.dataWeather.windDegName = 'Северный'
      } else if (deg === 90){
        this.dataWeather.windDegName = 'Западный'
      } else if (deg < 360 && deg > 270){
        this.dataWeather.windDegName = 'Юго-восточный'
      } else if (deg < 270 && deg > 180){
        this.dataWeather.windDegName = 'Северо-восточный'
      } else if (deg < 180 && deg > 90){
        this.dataWeather.windDegName = 'Северо-западный'
      } else if (deg < 90 && deg > 0){
        this.dataWeather.windDegName = 'Юго-западный'
      }
    },
    logoWeather() {
      let type = this.dataWeather.cloudsType;
      if (type === 'Clouds'){
        this.dataWeather.logo = './assets/cloud.png'
      } else if (type === 'Clear') {
        this.dataWeather.logo = './assets/sun.png'
      }
    },
    geoUser() {
      navigator.geolocation.getCurrentPosition((position) => {
        let lon = Math.trunc(position.coords.longitude);
        let lat = Math.trunc(position.coords.latitude);
        axios.get(`http://api.openweathermap.org/data/2.5/weather?lat=` + lat + `&lon=` + lon + `&appid=e275f27c1a9f13ff0b3f1746b12fd26c`).then(res => {
          this.nameTown = res.data.name;
          this.getData();
        })
      })
    },
    changeUnits(){
      if(this.units === 'metric'){
        this.units = 'imperial';
        this.getData();
      } else {
        this.units = 'metric';
        this.getData();
      }


    }
  },
  mounted() {
    this.getData();
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #ffffff;
  background: #498CEC;
  height: 100vh;
  overflow: hidden;
  padding: 50px;
  display: flex;
  justify-content: space-between;
  flex-direction: column;
}
.btn_ok{
  background: transparent;
  color: #009BFF;
  border: none;
  margin-left: -40px;
  outline: none;
}
.find_input {
  background: transparent;
  color: #ffffff;
  border: none;
}
  .main_weather > h1 {
    font-size: 115px;
  }
.main_weather img {
  width: 140px;
  height: 140px;
}
.main_weather span {
  font-size: 20px;
  margin-left: 15px;
}
</style>
