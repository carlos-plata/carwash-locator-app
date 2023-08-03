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
		this.findPlaces();
	}, methods: {
		initMap() {
			const loader = new Loader({
				apiKey: `${process.env.VUE_APP_GOOGLEMAPS_KEY}`,
				version: "weekly",
			});
			loader.load().then(async () => {
				const { Map } = await window.google.maps.importLibrary("maps");
				this.map = new Map(document.getElementById("map"), {
					center: { lat: this.latitude, lng: this.longitude },
					zoom: 14,
					mapTypeId: "roadmap",
				});
			});
		},
		async findPlaces(map) {
			const request = {
				query: "pharmacy",
				fields: ["name", "geometry"],
			};

			const { PlacesService } = await window.google.maps.importLibrary("places");
			const service = new PlacesService(new window.google.maps.Map(document.getElementById("map")));
			service.textSearch(request, (results, status) => {
				if (status === window.google.maps.places.PlacesServiceStatus.OK && results) {
					for (let i = 0; i < results.length; i++) {
						if (!results[i].geometry || !results[i].geometry.location) return;
						const marker = new window.google.maps.Marker({
							map: this.map, position: results[i].geometry.location,
						});
						window.google.maps.event.addListener(marker, "click", () => {
							window.google.maps.infowindow.setContent(results[i].name || "");
							window.google.maps.infowindow.open(map);
						});
					}
				}
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
