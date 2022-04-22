<template>
  <div class="relative flex flex-row justify-center items-center h-screen">
    <img
      :src="state.image"
      class="absolute top-0 left-0 w-full h-screen object-cover"
    />
    <div class="absolute w-full lg:w-1/4">
      <div class="rounded-lg bg-white bg-opacity-80 px-4 py-4">
        <div class="flex flex-row justify-center items-center space-x-3">
          <input
            type="text"
            v-model="state.country"
            @keydown.enter="refreshWeather"
            class="px-2 py-1 bg-transparent border-b border-b-gray-500"
          />
          <button type="button" @click="refreshWeather">
            <svg
              class="w-6 h-6 text-gray-500"
              fill="none"
              stroke="currentColor"
              viewBox="0 0 24 24"
              xmlns="http://www.w3.org/2000/svg"
            >
              <path
                stroke-linecap="round"
                stroke-linejoin="round"
                stroke-width="2"
                d="M21 21l-6-6m2-5a7 7 0 11-14 0 7 7 0 0114 0z"
              ></path>
            </svg>
          </button>
        </div>
      </div>

      <div class="rounded-lg bg-white bg-opacity-80 px-4 py-4 mt-6">
        <div v-if="state.weatherData" class="flex flex-col items-center mt-6">
          <p class="text-xl">{{ state.weatherData.location.name }}</p>
          <span class="text-5xl font-semibold mt-4"
            >{{ state.weatherData.current.temp_c }} <sup>°C</sup></span
          >
          <img
            :src="state.weatherData.current.condition.icon"
            class="mt-2 w-24"
          />
          <p class="uppercase">
            {{ state.weatherData.current.condition.text }}
          </p>
        </div>
      </div>
    </div>

    <!-- Error Messages -->
    <alert-message v-if="state.error" />
  </div>
</template>

<script setup>
import AlertMessage from "../components/AlertMessage.vue";
import { createApi } from "unsplash-js";
import { ref } from "@vue/reactivity";
import { onMounted, watch } from "@vue/runtime-core";
import axios from "axios";

const state = ref({
  image: null,
  country: "Beirut",
  weatherData: null,
  error: false
});

var accessKey = import.meta.env.VITE_UNSPLASHKEY;

onMounted(async () => {
  getImage();

  getWeather();
});

const getImage = async () => {
  //prepare Unsplash API connection
  const browserApi = createApi({
    accessKey: import.meta.env.VITE_UNSPLASH_KEY,
  });

  //calling Unsplash API To get the image
  var newImage = await browserApi.search.getPhotos({
    query: state.value.country,
    page: 1,
    perPage: 1,
  });

  //display image
  state.value.image = newImage.response.results[0].links.download;
};

const getWeather = async () => {
  //prepare Weather API connection
  const options = {
    method: "GET",
    url: "https://weatherapi-com.p.rapidapi.com/current.json",
    params: { q: state.value.country },
    headers: {
      "X-RapidAPI-Host": import.meta.env.VITE_WEATHER_HOST,
      "X-RapidAPI-Key": import.meta.env.VITE_WEATHER_KEY,
    },
  };

  try {
    let res = await axios.request(options);
    state.value.weatherData = res.data;
  } catch (e) {
      state.value.error = true;
    console.log(e.response);
  }
};

const refreshWeather = () => {
  getWeather();
  getImage();
};
</script>
