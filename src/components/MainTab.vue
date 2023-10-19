<script setup>
import { ref, onMounted, computed } from 'vue';
import WeatherInCity from "./WeatherInCity.vue";
import Graph from "./Graph.vue";
import GraphFiveDays from "./GraphFiveDays.vue";
import CloseButton from "./CloseButton.vue";
import cities from "../utils/cities.json";
import { API_KEY, BASE_URL, IP_KEY, IP_URL } from '../constans'

const startCity = 'Reykjavik';
const searchCity = ref(startCity);
const weatherInfo = ref(null);

async function getCustomerCity() {
  await fetch(`${IP_URL}?api-key=${IP_KEY}`)
    .then((response) => response.json())
    .then((data) => searchCity.value = data.city ? data.city : startCity)

    getWeather()
}

function getWeather()  {
  
  fetch(`${BASE_URL}?q=${searchCity.value}&appid=${API_KEY}&units=metric`)
    .then((response) => response.json())
    .then((data) => weatherInfo.value = data);

    searchCity.value = ''
};

const showListDialog = () => {
  if (weatherArray.value.length === 5) {
    dialogVisible.value = true;
  }
};

const filteredCities = computed(() => {
  return cities
    .filter((city) =>
        city.name.toLowerCase().startsWith(searchCity.value.toLowerCase())
    )
    .slice(0, 10);
});
  
function filterCities() {
  filteredCities.value;
};

const weatherArray = ref([]);

function updateWeatherArray(weatherInfo) {
  if (weatherInfo.list[0].weather && weatherArray.value.length < 5) {
      weatherArray.value.push(weatherInfo);
    }
};

let favoriteArray = ref([]);

function updateFavoriteArray(weatherInfo) {
  if (weatherInfo.list[0].weather) {
    // favoriteArray.value = JSON.parse(localStorage.getItem('favoriteArray'));

    if (favoriteArray.value.length < 5) {
        favoriteArray.value.push(weatherInfo);
        localStorage.setItem('favoriteArray', JSON.stringify(favoriteArray.value));
    }
  }
};

const dialogVisible = ref(false);

const showDialog = () => {
  if (favoriteArray.value && favoriteArray.value.length === 5) {
    dialogVisible.value = true;
  }
  else {
    return;
  }
};

const hideDialog = () => {
  dialogVisible.value = false;
};

const currentTab = ref("Day");

const currentListTab = ref("Day0");

onMounted(getCustomerCity)
onMounted(getWeather)
</script>

<template>
   <div class="container">
    <div class="container">
      <section class="inputSection">
        <div>
          <input
            id="inputMain"
            type="text"
            placeholder="Enter city" 
            v-model="searchCity" 
            @input="filterCities" 
            @keyup.enter="() => { 
                getWeather(); 
                searchCity = '';
            }"
          />
        </div>
      </section>
        
      <ul
        v-if="searchCity.length"
        onclick="this.style.display='none'"
      >
        <li 
          v-for="(city, index) in  filteredCities" 
          :key="index"
          @click="() => { searchCity = city.name; getWeather(); }"
        >
          {{ city?.name }}, {{ city?.country }}
        </li>
      </ul>
    </div>
    <section>
      <div v-if="!weatherInfo?.list" class="">
        City not found
      </div>
    </section>

    <section class="section">
      <div class="tabs">
        <button :class="{ active: currentTab === 'Day' }" @click="currentTab = 'Day'">
          Day
        </button>
        
        <button :class="{ active: currentTab === 'FiveDays' }" @click="currentTab = 'FiveDays'">
          5 Days
        </button>
      </div>

      <WeatherInCity v-if="weatherInfo?.list" :weatherInfo="weatherInfo" />

      <div v-if="currentTab === 'Day'" class="contentTab">
        <Graph v-if="weatherInfo?.list" :weatherInfo="weatherInfo" :index="2" />
      </div>
      <div v-if="currentTab === 'FiveDays'" class="contentTab">
        <GraphFiveDays v-if="weatherInfo?.list" :weatherInfo="weatherInfo" :index="3" />
      </div>
    </section>

    <button 
      v-if="weatherInfo?.list"
      class="btn addBtn" 
      @click="() => { updateWeatherArray(weatherInfo); showListDialog(); }"
    >
      Add to List
    </button>

    <button
      class="btn favorite"
      @click="() => { updateFavoriteArray(weatherInfo); showDialog(); }"
    >
      Add to Favorite
    </button>

    <div v-if="dialogVisible" class="cover-div"></div>
    <div v-if="dialogVisible" class="modal-div">
      <button @click="hideDialog">X</button>
      <p class="text content">To add new remove the city … because the maximum is 5</p>
    </div>
    
    <ul v-if="weatherArray.length > 0">
      <div class="title">List of weather in cities</div>
      <li 
        v-for="(cityWeather, index) in weatherArray" 
        :key="index"
      >
        <section class="section">
          <CloseButton :weatherArray="weatherArray" :index="index" />

          <div class="tabs">
            <button :class="{ active: currentListTab === `Day${index}` }" @click="currentListTab = `Day${index}`">
              Day
            </button>
            
            <button :class="{ active: currentListTab === `FiveDays${index}` }" @click="currentListTab = `FiveDays${index}`">
              5 Days
            </button>
          </div>

          <WeatherInCity v-if="weatherInfo?.list" :weatherInfo="cityWeather" />
          
          <div v-if="currentListTab === `Day${index}`" class="contentTab">
            <Graph v-if="weatherInfo?.list" :weatherInfo="cityWeather" :index="6 + index" />
          </div>
          <div v-if="currentListTab === `FiveDays${index}`" class="contentTab">
            <GraphFiveDays v-if="weatherInfo?.list" :weatherInfo="cityWeather" :index="7 + index" />
          </div>
        </section>
      </li>
    </ul>
  </div>
</template>

<style >
#inputMain {
  margin: 10px 0 0 10px;
}

.title {
  position: relative;
  top: 0;
  left: 0;
  margin: 10px;
}

.text {
  margin: 10px;
}
</style>
