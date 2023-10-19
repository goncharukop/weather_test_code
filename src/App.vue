<script setup>
import { ref, computed, } from 'vue';
import Main from "./components/MainTab.vue";
import Favorite from "./components/FavoriteTab.vue";

const currentTab = ref("Main");

const tabComponentName = {
  Main: Main,
  Favorite: Favorite,
};

const currentTabComponent = computed(() => tabComponentName[currentTab.value]);
</script>

<template>
  <div class="app">
    <div class="logo" style="background-image: url('/logo.png')"></div>
    <div class="tabs">
      <button :class="{ active: currentTab === 'Main' }" @click="currentTab = 'Main'">
        Main
      </button>
      <button :class="{ active: currentTab === 'Favorite' }" @click="currentTab = 'Favorite'">
        Favorite
      </button>
    </div>
    <div class="contentTab">
      <component :is="currentTabComponent"></component>
    </div>
  </div>
</template>

<style>
.tabs {
  display: flex;
  position: relative;
}

.tabs button {
  padding: 15px;
  border-top-right-radius: 10px;
  border-top-left-radius: 10px;
  font-size: 22px;
  background-color: lightgray;
}

.tabs button.active {
  background-color: #2da9fc;
  color: white;
}

.contentTab {
  border: 2px solid white;
  background-color: #655bfa;
}

.logo {
    width: 350px;
    height: 120px;
    margin: auto;
    background-repeat: no-repeat;
    background-position: 50% 50%;
    background-size: contain;
}
</style>
