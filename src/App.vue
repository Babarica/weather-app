 <template>
  <div id="#app">
     <aside class="sidebar" :class="sidebar">
        <nav v-if="cities.length > 0">
          <a class="closeBtn" :class="close" @click="toggleClose"></a>
          <div v-for="town in cities" :key="town.id">
              <div class="side-town">
                <div class="side-temp" @click="findCity(town.city)">
                  <img :src="showWeather(town.weather)" alt="">
                  <div class="side-weather">
                    <p class="side-weather__temp">{{ town.temp }}°c</p>
                    <p>{{ town.weather }}</p>
                  </div>
                </div>
                <div class="side-group">
                  <p>{{ town.city }}</p><div @click="removeCity(town)" class="side-close"></div>
                </div>
              </div>
          </div>
        </nav>
        <div class="side-empty" v-else>
          <h1>List of cities is empty</h1>
          <a class="closeBtn" :class="close" @click="toggleClose"></a>
        </div>
      </aside>
      <a class="close" :class="close" @click="toggleClose"></a>
    <div class="container">
        <div class="header-box">
          <h1 class="headerTitle">WIYС</h1>
          <div class="header-group">
            <div class="header-burger" :class="close" @click="togglemenu">
              <span></span>
              <span></span>
              <span></span>
            </div>
            <div class="header-inner">
              <a @click="changeWeatherMouse" class="header-search"></a>
              <input 
              v-focus
              class="header-input" 
              type="text" 
              placeholder="search..."
              v-model="query"
              @keypress="changeWeather"
              >
            </div>
            <button class="header-button" @click="addCity()"></button>
          </div>
      </div>
      <main class="main">
        <div class="main-search" v-show="active">
          <p class="main-city">Enter city...</p>
          <img class="main-town" src="./assets/Town.svg" alt="">
        </div>
        <transition name="made">
          <div class="main-box" v-if="typeof weather.list != 'undefined'">
            <div class="main-img">
              <img src="./assets/cloudy.png" alt="">
            </div>
            <div class="main-group higher">
              <p class="main-temp">{{ Math.floor(weather.list[0].main.temp)}}°c</p>
              <p class="main-date">{{dateBuilder()}}</p>
            </div>
            <div class="main-group">
              <p class="main-location">{{weather.city.name}}</p>
              <div class="main-block">
                <div class="main-section">
                  <p>weather:</p>
                  <p>wind:</p>
                  <p>pressure:</p>
                  <p>visibility:</p>
                </div>
                <div class="main-section">
                  <p>{{weather.list[0].weather[0].main}}</p>
                  <p>{{Math.floor(weather.list[0].wind.speed)}} m/s</p>
                  <p>{{weather.list[0].main.pressure}} hPa</p>
                  <p>{{weather.list[0].visibility/1000}} km</p>
                </div>
              </div>
            </div>
          </div>
        </transition>
      </main>
      <section class="week">
        <transition-group name='list' tag="p">
          <div v-for="(day, index) in score" :key="day" class="list-item">
            <div class="day" v-if="day.dt_txt !== weather.list[0].dt_txt">
              <div class="day-weather">
                <div class="day-block">
                  <div class="day-group">
                    <img :src="showWeather(day.weather[0].main)" alt="">
                    <p class="day-grad">{{day.weather[0].main}}</p>
                  </div>
                  <p class="day-temp">{{Math.floor(day.main.temp)}}°c</p>
                  <p class="night-temp">{{ Math.floor(setDay(index)) }}°c</p>
                </div>
                <div class="day-weth">{{ getWeather(index) }}</div>
              </div>
            </div>
          </div>
        </transition-group>
      </section>
    </div>
  </div>
</template>


<script>
  
export default {
  name: 'App',
  data(){
    return {
      api_key: 'ef1997cc54103c25f43ed4a7059f612c',
      url_base: 'https://api.openweathermap.org/data/2.5/',
      query: '',
      weather: {},
      day: {
        city: '',
        temp: 0,
        weather: '',
      },
      active: true,
      score: [],
      nightDate: [],
      smallDays: ["Sun", "Mon", "Tue", "Wedn", "Thur", "Fr", "Sat"],
      days: ["Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday"],
      NewDay: new Date(),
      cities: [],
      querySide: '',
      sidebar: {
        active: false,
      },
      headerTitle: {
        opac: false,
        non: false,
        tran: false,
      },
      close: {
        active: false,
        activeTrig: false,
      },
      headerGroup: {
        tran: false
      } 
      
    }
  },
  methods: {
    findCity(town) {
      this.query = town;
      fetch(`${this.url_base}forecast?q=${this.query}&units=metric&APPID=${this.api_key}`,{method: 'GET',}).then(res=>{
          return res.json();
        }).then(this.setResults);
      this.sidebar.active = false;
       this.active = false;
      this.close.active = false;
    },
    addCity(){
      this.day.id = Date.now();
      this.day = {
        city: this.weather.city.name,
        temp: Math.floor(this.weather.list[0].main.temp),
        weather: this.weather.list[0].weather[0].main,
      }
      this.cities.push(this.day);
      this.close.activeTrig = true;
      this.setLocalTowns();

    },
    getWeather(index){
      let score = this.NewDay.getDay();
      if(window.innerWidth < 425){
         if(score == 6){
          score = 0;
          return this.smallDays[score+(index)];
          }
         if(score+index >= 6){
          switch (index){
            case 1:
            return this.smallDays[score];
            case 2: {
            return this.smallDays[0];
            }
            case 3: {
            return this.smallDays[1];
            }
            case 4: {
            return this.smallDays[2];
            }
          }
        }
        else {
          return this.smallDays[score+(index+1)];
        }
      }
      else {
        if(score == 6){
          score= 0;
         return this.days[score+(index)];
        }
        if(score+index >= 6){
          switch (index){
            case 1:
            return this.days[score];
            case 2: {
            return this.days[0];
            }
            case 3: {
            return this.days[1];
            }
            case 4: {
            return this.days[2];
            }
          }
        }
        else {
          return this.days[score+(index+1)];
        }
      }
      
      
    },
    setDay(index){
      let s = index + 1;
      if (s == 5 && this.nightDate[5] != 'undefined'){
        return this.nightDate[index].main.temp;
      }
      return this.nightDate[s].main.temp;
    },
    removeCity(town){
      this.cities = this.cities.filter(c => c.id !== town.id);
      this.setLocalTowns();
    },
    togglemenu(){
      this.sidebar.active = true;
      this.close.active = true;
      this.headerTitle.opac = true;
      setTimeout(()=>{
        this.headerTitle.non = true;
      }, 2000);
      this.headerGroup.tran = true
    },
    toggleClose(){
      this.sidebar.active = false;
      this.close.active = false;
    },
    showWeather(day){
      switch (day){
        case "Rain":
          return require('./assets/rain.svg');
        case "Clear":
          return require('./assets/sun.svg');
        case "Clouds":
          return require('./assets/cloud.svg');
        case "Snow":
          return require('./assets/snow.svg')
      }
    },
    changeWeatherMouse(){
      this.active = false;
        fetch(`${this.url_base}forecast?q=${this.query}&units=metric&APPID=${this.api_key}`,{method: 'GET',}).then(res=>{
        return res.json();
      }).then(this.setResults);
    },
    changeWeather(e){
      if(e.key == "Enter"){
        this.active = false;
        fetch(`${this.url_base}forecast?q=${this.query}&units=metric&APPID=${this.api_key}`,{method: 'GET',}).then(res=>{
          return res.json();
        }).then(this.setResults);
      }
    },
    setResults(results){
      this.score = results.list;
      let scoreDate = [];
      let nightTemp = [];
      for(let i  =0; i< this.score.length; i++){
        let m = this.score[i].dt_txt;
        m = m.split(' ');
        if(m[1] === '12:00:00'){
          scoreDate.push(this.score[i]);
        }
      }
      for(let i  =0; i< this.score.length; i++){
        let m = this.score[i].dt_txt;
        m = m.split(' ');
        if(m[1] === '21:00:00'){
          nightTemp.push(this.score[i]);
        }
      }
      this.nightDate = nightTemp;
      this.score = scoreDate;
      this.weather = results;
      console.log(this.weather);
    },
     dateBuilder () {
       let d = new Date();
      let months = ["January", "February", "March", "April", "May", "June", "July", "August", "September", "October", "November", "December"];
      var days = ["Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday"];
      let day = days[d.getDay()];
      let date = d.getDate();
      let month = months[d.getMonth()];
      return `${day} ${date} ${month}`;
    },
    setLocalTowns(){
      // localStorage.cities = this.cities
      const parsed = JSON.stringify(this.cities);
      localStorage.setItem('towns', parsed);
    }

  },
  mounted(){
    if (localStorage.getItem('towns')){
        try{
          this.cities = JSON.parse(localStorage.getItem('towns'));
        } catch(e) {
          localStorage.removeItem('towns')
        }
      }
    }
      
}
</script>

<style>
  .header-button::after {
    background-image: url(./assets/butCity.png);
  }
  .header-search {
    background-image: url(./assets/search.svg);
  }
  .side-search {
    background-image: url(./assets/search.svg);
  }
  .side-close {
     background-image: url(./assets/close.png);
  }
  .closeBtn {
    background-image: url(./assets/close.png);
  }
  .main-box::after {
    background: url(./assets/wawes.png)
  }
    @import url(./assets/style/app.css);
</style>
