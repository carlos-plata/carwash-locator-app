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
			directionsRenderer: null,
		}
	},
	mounted() {
		this.$nextTick(() => {
			this.initMap();
			this.findPlaces();
		});
	}, updated() {
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
					gestureHandling: 'cooperative', // Default behavior
				});
			});
		},
		async findPlaces() {
			const request = {
				query: "pharmacy",
				fields: ["name", "geometry", "place_id"], // Include the fields you need
			};

			const { PlacesService } = await window.google.maps.importLibrary("places");
			const service = new PlacesService(new window.google.maps.Map(document.getElementById("map")));
			let currentInfoWindow = null;

			service.textSearch(request, (results, status) => {
				if (status === window.google.maps.places.PlacesServiceStatus.OK && results) {
					for (let i = 0; i < results.length; i++) {
						if (!results[i].geometry || !results[i].geometry.location) return;

						const placeId = results[i].place_id;
						const marker = new window.google.maps.Marker({
							map: this.map,
							position: results[i].geometry.location,
						});

						const infoWindow = new window.google.maps.InfoWindow();
						infoWindow.isOpen = false;  // Custom attribute to track the state of the infoWindow


						window.google.maps.event.addListener(marker, "mouseover", () => {
							if (infoWindow.isOpen) return; // If the infoWindow is already open, just return
							// Only fetch details when the marker is hovered over
							const placeRequest = {
								placeId: placeId,
								fields: ["name", "formatted_address", "geometry", "rating", "user_ratings_total", "opening_hours", "photo"], // Request only the fields needed
							};

							service.getDetails(placeRequest, (place, status) => {
								console.log(place.opening_hours);
								if (status === window.google.maps.places.PlacesServiceStatus.OK) {
									if (currentInfoWindow) {
										currentInfoWindow.close();
										currentInfoWindow.isOpen = false;  // Update state of the previous infoWindow
									}

									const destination = place.geometry.location;
									let openingHoursText = 'Opening hours unavailable';
									let weeklyHours = '';

									if (place.opening_hours) {
										const today = new Date().getDay() - 1;

										// Construct the weekly hours text
										weeklyHours = place.opening_hours.weekday_text.map((dayText) => {
											return dayText;
										}).join('<br>'); // Join each day's text with a break line

										// Construct the current day's hours text
										openingHoursText = place.opening_hours.weekday_text[today];
										if (place.opening_hours.open_now) {
											openingHoursText += ' (Open now)';
										} else {
											openingHoursText += ' (Closed)';
										}
									}
									const rating = place.rating ? `${place.rating} - (${place.user_ratings_total} ratings)` : '';
									let photoUrl = '';
									if (place.photos && place.photos.length > 0) {
										photoUrl = place.photos[0].getUrl({ maxWidth: 100 });
									}
									const contentString = `<div class="infowindow-content"><img src="${photoUrl}" alt="${place.name}" class="infowindow-image" /><br><h5>${place.name || ""}</h5><strong>Address: </strong><br>${place.formatted_address || ""}<div><strong>${openingHoursText}</strong></div><div><br><strong>Business Hours: </strong><br>${weeklyHours}</div><div><br><strong>Ratings: </strong>${rating}</div></div>`;
									const contentElement = document.createElement('div');
									contentElement.innerHTML = contentString;
									const link = document.createElement('a');
									link.href = "javascript:void(0);";
									link.textContent = "Get Directions";
									link.addEventListener('click', () => {
										this.getDirections(destination);
										infoWindow.close(); // Close the info window when clicking "Get Directions"
									});
									contentElement.appendChild(link);
									infoWindow.setContent(contentElement);
									infoWindow.open(this.map, marker);
									infoWindow.isOpen = true;  // Update the state to open
									currentInfoWindow = infoWindow;
								}
							});
						});
						window.google.maps.event.addListener(infoWindow, 'closeclick', function () {
							infoWindow.isOpen = false;  // Update the state when the infoWindow is manually closed
						});
					}
				}
			});
		},
		getDirections(destination) {
			if (navigator.geolocation) {
				navigator.geolocation.getCurrentPosition((position) => {
					// If there is an existing directionsRenderer, remove it from the map
					if (this.directionsRenderer) {
						this.directionsRenderer.setMap(null);
					}
					this.directionsRenderer = new window.google.maps.DirectionsRenderer(); // Instantiate new Directions Renderer
					this.directionsRenderer.setMap(this.map);

					const request = {
						origin: {
							lat: position.coords.latitude,
							lng: position.coords.longitude,
						},
						destination: destination,
						travelMode: 'DRIVING',
					};

					const directionsService = new window.google.maps.DirectionsService(); // Instantiate Directions Service
					directionsService.route(request, (result, status) => {
						if (status === 'OK') {
							this.directionsRenderer.setDirections(result);
						}
					});
				}, () => {
					alert('Error fetching your location. Please allow location access.');
				});
			} else {
				alert('Geolocation is not supported by this browser.');
			}
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
}

.infowindow-content {
	width: 300px;
	/* Adjust this as needed */
}

.infowindow-image {
	width: 100%;
	height: auto;
	max-height: 200px;
	/* Adjust this as per your requirement */
	display: block;
	margin: 0 auto;
}
</style>
