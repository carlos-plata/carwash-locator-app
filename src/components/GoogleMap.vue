<template>
	<div id="map"></div>
</template>

<script>
//importing components
import { Loader } from "@googlemaps/js-api-loader";
//App definition
export default {
	name: 'GoogleMap',
	components: {
	},
	props: {
		city: String,
		latitude: Number,
		longitude: Number,

	},
	data() {
		return {
			map: null,
		}
	},
	mounted() {
		this.initMap();
	}, methods: {
		initMap() {
			console.log("Latitude : " + this.latitude + " Longitude: " + this.longitude + ", CITY: " + this.city);
			const loader = new Loader({
				apiKey: "AIzaSyBMy8bWA8Bielkpn5-mLxnIpt683DZrHUs",
				version: "weekly",
			});
			loader.load().then(async () => {
				const { Map } = await window.google.maps.importLibrary("maps");
				this.map = new Map(document.getElementById("map"), {
					center: { lat: this.latitude, lng: this.longitude },
					zoom: 16,
					mapTypeId: "roadmap",
				});
			});
		}
	}
}
</script>

<style>
/* 
 * Always set the map height explicitly to define the size of the div element
 * that contains the map. 
 */
#map {
	height: 100%;
}

/* 
 * Optional: Makes the sample page fill the window. 
 */
html,
body {
	background-color: #121212 !important;
	height: 100%;
	margin: 0;
	padding: 0;
}

:root {
	--mdc-theme-primary: #1a73e8;
}

#wrapper {
	width: 100%;
	height: 100%;
	display: flex;
	align-items: center;
	justify-content: center;
	background-size: cover;
}
</style>