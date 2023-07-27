<template>
	<div class="container p-0">
		<div class="d-flex">
			<div class="card main-div w-100">
				<div class="p-3">
					<h2 class="mb-1 day">Today</h2>
					<p class="text-light date mb-0">{{ date }}</p>
					<small>{{ time }}</small>
					<h2 class="place"><i class="fa fa-location">{{ name }} <small>{{ country }}</small></i></h2>
					<div class="temp">
						<h1 class="weather-temp">{{ temperature }}&deg;</h1>
						<h2 class="text-light">{{ description }} <img :src="iconUrl"></h2>
					</div>
				</div>
			</div>
			<div class="card-2 w-100">
				<table class="m-4">
					<tbody>
						<tr>
							<th>Feels like:</th>
							<td>{{ feelsLike }}&deg;</td>
						</tr>
						<tr>
							<th>Min:</th>
							<td>{{ tempMin }}&deg;</td>
						</tr>
						<tr>
							<th>Max</th>
							<td>{{ tempMax }}&deg;</td>
						</tr>
					</tbody>
				</table>
				<ForecastWeather></ForecastWeather>
				<div class="d-flex m-3 justify-content-center" id="div_Form">
					<form action="">
						<input type="button" value="Change Location" class="btn change-btn btn-primary">
					</form>
				</div>
			</div>
		</div>
	</div>
</template>

<script>
import axios from 'axios';
//importing components
import ForecastWeather from './ForecastWeather.vue';

export default {
	name: 'WeatherComponent',
	components: {
		ForecastWeather
	},
	props: {
		city: String,

	},
	data() {
		return {
			temperature: null,
			feelsLike: null,
			tempMin: null,
			tempMax: null,
			description: null,
			iconUrl: null,
			date: null,
			time: null,
			name: null,
			country: null
		}
	},
	async created() {
		const response = await axios.get(`https://api.openweathermap.org/data/2.5/weather?q=${this.city}&units=metric&appid=${process.env.VUE_APP_OPENWEATHERMAP_KEY}`);
		const weatherData = response.data;
		this.temperature = Math.round(weatherData.main.temp);
		this.feelsLike = Math.round(weatherData.main.feels_like);
		this.tempMin = Math.round(weatherData.main.temp_min);
		this.tempMax = Math.round(weatherData.main.temp_max);
		this.description = weatherData.weather[0].description;
		this.iconUrl = `https://api.openweathermap.org/img/w/${weatherData.weather[0].icon}.png`;
		this.date = new Date().toLocaleDateString();
		const d = new Date();
		this.time = d.getHours() + ':' + d.getMinutes() + ':' + d.getSeconds();
		this.name = weatherData.name;
		this.country = weatherData.sys.country;
		console.log(weatherData);
	}
}
</script>

<style>
body {
	background-color: #343d4b;
}

.weather-temp {
	margin: 0;
	font-weight: 700;
	font-size: 4em;
}

h2.mb-1.day {
	font-size: 3rem;
	font-weight: 400;
}

.main-div {
	border-radius: 20px;
	color: #fff;
	background-image: url("../assets/weather.jpg");
	background-size: cover;
	background-position: center;
	/*background-blend-mode: overlay;*/
	background-color: rgba(0, 0, 0, 0.5);
	background-repeat: no-repeat;
}

.temp {
	position: absolute;
	bottom: 0;
}

.main-div:hover {
	transform: scale(1.1);
	transition: transform 0.5s ease;
	z-index: 1;
}

.card-2 {
	background-color: #212730;
	border-radius: 20px;
}

.card-details {
	margin-left: 19px;
}

.h1_left {
	position: absolute;
	bottom: 25px;
	left: 16px;
	font-size: 3vw;
	line-height: 1.2;
}

.h3_left {
	position: absolute;
	left: 16px;
	font-size: 2vw;
	line-height: 0.5;
}

.h3_left small {
	font-size: 1rem;
}

table {
	position: relative;
	left: 15px;
	border-collapse: separate;
	border-spacing: 15px;
	width: 85%;
	text-align: left;
	max-width: 600px;
	margin: 0 auto;
}

th,
td {
	font-size: 18px;
	color: #fff;

}

td {
	text-align: right;
}

table,
tr:hover {
	color: red;
}

.change_btn {
	background-image: linear-gradient(to right, lavender, teal);
}

.change_btn:hover {
	transform: scale(0.9);
	transition: transform 0.1s ease;
}
</style>
