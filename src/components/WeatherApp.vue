<script setup lang="ts">
import { onMounted, reactive } from "vue";
import axios from "axios";
const openWeatherMapKey = "dc78a70f3f9ef9674fe6a80456a810b3";

interface State {
    query: string;
    weather: Record<string, any>;
    error: boolean;
    location: {
        latitude: number | null;
        longitude: number | null;
    };
    weather_bg: string
}

const state = reactive<State>({
    query: "",
    weather: {},
    error: false,
    location: {
        latitude: null,
        longitude: null,
    },
    weather_bg: "sunny",
});

const fetchWeather = async (e: Event) => {
    const input = (e.target as HTMLInputElement).value;
    try {
        const response = await axios.get(
            `https://api.openweathermap.org/data/2.5/weather?q=${encodeURIComponent(
                input
            )}&units=metric&appid=${openWeatherMapKey}`
        );
        state.weather = response.data;
        state.error = false;
        state.weather_bg = changeBG();
    } catch (error) {
        (state.weather = {}), (state.error = true);
    }
};

const changeBG = () => {
    if (typeof state.weather.main !== "undefined") {
        const weatherType = state.weather.weather[0]?.main.toLowerCase();
        const temp = state.weather.main.temp;

        if (weatherType === "rain" || weatherType === "snow") {
            return weatherType;
        } else if (temp < 10) {
            return "cold";
        } else if (temp < 25) {
            return "warm";
        } else {
            return "sunny";
        }
    }
    return "sunny";
};

const dateBuilder = () => {
    const days = [
        "Sunday",
        "Monday",
        "Tuesday",
        "Wednesday",
        "Thursday",
        "Friday",
        "Saturday",
    ];
    const months = [
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
    const d = new Date();
    const day = days[d.getDay()];
    const month = months[d.getMonth()];
    const date = d.getDate();
    const year = d.getFullYear();
    return `${day}, ${month} ${date}, ${year}`;
};

const fetchWeatherByLocation = async (lat: number, lon: number) => {
    try {
        const encodedLat = encodeURIComponent(lat);
        const encodedLon = encodeURIComponent(lon);

        const response = await axios.get(
            `https://api.openweathermap.org/data/2.5/weather?lat=${encodedLat}&lon=${encodedLon}&units=metric&appid=${openWeatherMapKey}`
        );

        // Update state with weather data
        state.weather = response.data;
        state.error = false;
    } catch (error) {
        console.error('Error fetching weather data:', error);
        state.error = true; // Set error flag in state
    }
};

const getUserLocation = () => {
    if (navigator.geolocation) {
        navigator.geolocation.getCurrentPosition(
            (position) => {
                const { latitude, longitude } = position.coords;
                state.location.latitude = latitude;
                state.location.longitude = longitude;

                // Fetch weather using the user's current location
                fetchWeatherByLocation(latitude, longitude);
            },
            (error) => {
                console.error("Error getting location: ", error);
                state.error = true;
            }
        );
    } else {
        console.error("Geolocation is not supported by this browser.");
        state.error = true;
    }
}

onMounted(() => {
    console.log('App Mounted.')
    getUserLocation();
});
</script>

<template>
    <div id="main-app" :class="state.weather_bg">
        <main>
            <div class="w-100 text-center">
                <input type="text" class="mt-20 w-1/2 offset-3 p-2 rounded" v-model="state.query" @keyup="fetchWeather"
                    placeholder="Search for a city" />
            </div>
            <div class="weather-wrap">
                <div class="location-box">
                    <div class="location">
                        <span v-if="state.error">City not found</span>
                        <span v-else-if="!state.weather.main">Search for a city</span>
                        <span v-else>{{ state.weather.name }}, {{ state.weather.sys?.country }}</span>
                    </div>
                    <div class="date">
                        <span>{{ dateBuilder() }}</span>
                    </div>
                </div>
                <div class="weather-box" v-if="typeof state.weather.main !== 'undefined' && !state.error">
                    <div class="temp">
                        <span>{{ Math.round(state.weather.main.temp) }}°C</span>
                    </div>
                    <div class="weather">
                        <span>{{ state.weather.weather[0]?.main }}</span>
                    </div>
                </div>
            </div>
        </main>
    </div>
</template>

<style scoped>
#main-app {
    background-size: cover;
    background-position: bottom;
    transition: 0.4s;
}

#main-app.sunny {
    background-image: url("../assets/sunny-bg.jpg");
}

#main-app.cold {
    background-image: url("../assets/cold-bg.jpg");
}

#main-app.warm {
    background-image: url("../assets/warm-bg.jpg");
}

#main-app.rain {
    background-image: url("../assets/rain-bg.jpg");
}

main {
    width: 100%;
    min-height: 100vh;
    padding: 25px;
    background-image: linear-gradient(to bottom,
            rgba(0, 0, 0, 0.25),
            rgba(0, 0, 0, 0.75));
}

.location-box .location {
    color: #fff;
    font-size: 32px;
    font-weight: 500;
    text-align: center;
    text-shadow: 1px 3px rgba(0, 0, 0, 0.25);
}

.location-box .date {
    color: #fff;
    font-size: 20px;
    font-weight: 300;
    text-align: center;
    font-style: italic;
}

.weather-box {
    text-align: center;
}

.weather-box .temp {
    display: inline-block;
    padding: 10px 25px;
    color: #fff;
    font-size: 100px;
    font-weight: 900;
    text-shadow: 3px 6px rgba(0, 0, 0, 0.25);
    background-color: rgba(255, 255, 255, 0.25);
    border-radius: 16px;
    margin: 30px 0;
    box-shadow: 3px 6px rgba(0, 0, 0, 0.25);
}

.weather-box .weather {
    color: #fff;
    font-size: 48px;
    font-weight: 700;
    font-style: italic;
    text-shadow: 3px 6px rgba(0, 0, 0, 0.25);
}
</style>
