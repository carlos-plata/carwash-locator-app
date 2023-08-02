<template>
	<div class="app">
		<div class="header container h-100 p-5">
			<h1 class="mb-5">CarWash Cerca De Mi</h1>
			<div class="d-flex justify-content-center h-100">
				<div class="searchbar w-50 mx-2">
					<input type="text" class="input form-control" v-model="city" placeholder="Enter a City...">
				</div>
				<button class="btn-search btn btn-primary" @click="searchWeather">Search <i
						class="fas fa-search"></i></button>
			</div>
		</div>
		<div class="container p-0" v-if="showWeather">
			<div class="row">
				<div class="col">
					<GoogleMap :city="city" :latitude="latitude" :longitude="longitude"></GoogleMap>
				</div>
				<div class="col">
					<Weather :city="city" :latitude="latitude" :longitude="longitude"></Weather>
				</div>
			</div>
		</div>
	</div>
</template>

<script>
//importing components
import Weather from './components/Weather.vue'
import GoogleMap from './components/GoogleMap.vue'
//App definition
export default {
	name: 'App',
	components: {
		Weather,
		GoogleMap,
	},
	data() {
		return {
			city: '',
			showWeather: false,
			latitude: null,
			longitude: null,
		}
	},
	async created() {
		var geoOps = {
			enableHighAccuracy: true,
			timeout: 10000 //10 seconds
		}
		navigator.geolocation.getCurrentPosition(this.initPosition, null, geoOps);
	},
	methods: {
		async searchWeather() {
			this.showWeather = false;
			await this.$nextTick();
			this.showWeather = true;
		},
		initPosition(position) {
			this.latitude = position.coords.latitude;
			this.longitude = position.coords.longitude;
		},
	}
}
</script>

<style>
body {
	background-color: #121212 !important;
}

.header {
	background-color: #212730;
	border-radius: 20px;
	color: #fff;
	text-align: center;
	font-family: 'Franklin Gothic Medium', 'Arial Narrow', Arial, sans-serif;
	margin-top: 5rem;
}

.btn-search {
	background-image: linear-gradient(to right, lavender, teal);
}
</style>
