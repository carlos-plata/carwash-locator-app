<template>
	<div class="tooltip" ref="tooltip">Use two fingers to move the map</div>
	<div id="map"></div>
</template>

<script>
import { Loader } from "@googlemaps/js-api-loader";

export default {
	name: 'GoogleMap',
	props: {
		city: String,
		latitude: Number,
		longitude: Number,
	},
	data() {
		return {
			map: null,
			directionsRenderer: null,
			currentDestination: null,
		};
	},
	mounted() {
		this.$nextTick(() => {
			this.initMap();
			this.findPlaces();
		});
	},
	methods: {
		initMap() {
			if (this.map) return;

			const loader = new Loader({
				apiKey: `${process.env.VUE_APP_GOOGLEMAPS_KEY}`,
				version: "weekly",
				libraries: ["maps", "places"],
			});

			loader.load().then(() => {
				this.map = new window.google.maps.Map(document.getElementById("map"), {
					center: { lat: this.latitude, lng: this.longitude },
					zoom: 14,
					mapTypeId: "roadmap",
					gestureHandling: 'cooperative',
				});

				// Traffic layer
				const trafficLayer = new window.google.maps.TrafficLayer();
				trafficLayer.setMap(this.map);

				// Show the tooltip only on mobile devices (viewport width less than 768 pixels for instance)
				if (window.innerWidth < 768) {
					this.$refs.tooltip.style.opacity = "1";
					setTimeout(() => {
						this.$refs.tooltip.style.opacity = "0";
					}, 3000);
				}
			});
		},
		async findPlaces() {
			const request = {
				query: "pharmacy",
				fields: ["name", "geometry", "place_id"],
			};

			const { PlacesService } = await window.google.maps.importLibrary("places");
			const service = new PlacesService(new window.google.maps.Map(document.getElementById("map")));
			let currentInfoWindow = null;

			service.textSearch(request, (results, status) => {
				if (status === window.google.maps.places.PlacesServiceStatus.OK && results) {
					for (let i = 0; i < Math.min(results.length, 10); i++) {
						if (!results[i].geometry || !results[i].geometry.location) return;

						const placeId = results[i].place_id;
						const marker = new window.google.maps.Marker({
							map: this.map,
							position: results[i].geometry.location,
						});

						const infoWindow = new window.google.maps.InfoWindow();
						infoWindow.isOpen = false;

						window.google.maps.event.addListener(marker, "mouseover", () => {
							if (infoWindow.isOpen) return;

							const placeRequest = {
								placeId: placeId,
								fields: ["name", "formatted_address", "geometry", "rating", "user_ratings_total", "opening_hours", "photo"],
							};

							service.getDetails(placeRequest, (place, status) => {
								if (status === window.google.maps.places.PlacesServiceStatus.OK) {
									if (currentInfoWindow) {
										currentInfoWindow.close();
										currentInfoWindow.isOpen = false;
									}

									const destination = place.geometry.location;
									let weeklyHours = '';

									if (place.opening_hours) {
										weeklyHours = place.opening_hours.weekday_text.join('<br>');
									}

									const rating = place.rating ? `${place.rating} - (${place.user_ratings_total} ratings)` : '';
									let photoUrl = '';
									if (place.photos && place.photos.length > 0) {
										photoUrl = place.photos[0].getUrl({ maxWidth: 100 });
									}
									const contentString = `
									    <div class="infowindow-content">
									        <img src="${photoUrl}" alt="${place.name}" class="infowindow-image" />
									        <h5>${place.name || ""}</h5>
									        <div class="infowindow-section">
									            <i class="material-icons">location_on</i>
									            <strong>Address:</strong>
									            <p>${place.formatted_address || ""}</p>
									        </div>
									        <div class="infowindow-section">
									            <i class="material-icons">schedule</i>
									            <strong>Business Hours:</strong>
									            <p>${weeklyHours}</p>
									        </div>
									        <div class="infowindow-section">
									            <i class="material-icons">star_rate</i>
									            <strong>Ratings:</strong>
									            <p>${rating}</p>
									        </div>
									    </div>
									    <a href="javascript:void(0);" class="directions-link">Get Directions</a>
									`;

									const contentElement = document.createElement('div');
									contentElement.innerHTML = contentString;

									// Handle click event for the "Get Directions" link
									const directionsLink = contentElement.querySelector('.directions-link');
									if (directionsLink) {
										directionsLink.addEventListener('click', () => {
											this.getDirections(destination);
											infoWindow.close();
											console.log("Attaching listener to directions link.");
										});
									}
									infoWindow.setContent(contentElement);
									infoWindow.open(this.map, marker);
									infoWindow.isOpen = true;
									currentInfoWindow = infoWindow;
								}
							});
						});
						window.google.maps.event.addListener(infoWindow, 'closeclick', function () {
							infoWindow.isOpen = false;
						});
					}
				}
			});
		},
		getDirections(destination) {
			console.log("Get Directions clicked for destination:", destination);
			this.currentDestination = destination;
			const startIcon = {
				path: window.google.maps.SymbolPath.CIRCLE,
				fillColor: "#00FF00",
				fillOpacity: 1,
				scale: 6,
				strokeColor: "#FFFFFF",
				strokeWeight: 2
			};

			const endIcon = {
				path: window.google.maps.SymbolPath.CIRCLE,
				fillColor: "#FF0000",
				fillOpacity: 1,
				scale: 6,
				strokeColor: "#FFFFFF",
				strokeWeight: 2
			};

			if (!this.directionsRenderer) {
				this.directionsRenderer = new window.google.maps.DirectionsRenderer({
					polylineOptions: {
						strokeColor: '#4285F4',
						strokeOpacity: 0.8,
						strokeWeight: 5,
					},
				});
				this.directionsRenderer.setMap(this.map);
			}

			if (navigator.geolocation) {
				navigator.geolocation.getCurrentPosition((position) => {
					const request = {
						origin: {
							lat: position.coords.latitude,
							lng: position.coords.longitude,
						},
						destination: destination,
						travelMode: 'DRIVING',
					};

					const directionsService = new window.google.maps.DirectionsService();
					directionsService.route(request, (result, status) => {
						if (status === 'OK') {
							this.directionsRenderer.setDirections(result);
							// Remove default markers
							this.directionsRenderer.setOptions({
								suppressMarkers: true
							});

							// Add custom start marker
							new window.google.maps.Marker({
								position: result.routes[0].legs[0].start_location,
								map: this.map,
								icon: startIcon
							});

							// Add custom end marker
							new window.google.maps.Marker({
								position: result.routes[0].legs[0].end_location,
								map: this.map,
								icon: endIcon
							});

						}
					});
				}, () => {
					alert('Error fetching your location. Please allow location access.');
				});
			} else {
				alert('Geolocation is not supported by this browser.');
			}
		},
	}
}
</script>

<style>
#map {
	height: 100%;
}

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

.infowindow-container {
	display: flex;
	flex-direction: column;
	background-color: #fff;
	box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
	border-radius: 8px;
	overflow: hidden;
}

.infowindow-image {
	width: 100%;
	max-height: 150px;
	object-fit: cover;
}

.infowindow-content {
	padding: 12px 16px;
}

.infowindow-title {
	margin: 0;
	color: #333;
	font-size: 1.1em;
	font-weight: bold;
}

.infowindow-address {
	display: flex;
	align-items: center;
	margin-top: 2px;
	color: #777;
	font-size: 0.9em;
}

.infowindow-hours {
	display: flex;
	align-items: center;
	margin-top: 2px;
	color: #555;
	font-size: 0.9em;
}

.infowindow-rating {
	display: flex;
	align-items: center;
	margin-top: 2px;
	color: #555;
	font-size: 0.9em;
}

.infowindow-section {
	margin-bottom: 2px;
}

.infowindow-section p {
	margin-top: 2px;
}

.material-icons {
	margin-right: 8px;
	font-size: 18px;
	color: #1a73e8;
}

.directions-link {
	display: block;
	margin-top: 15px;
	/* Add a margin to the top to push it further down */
	text-align: center;
	background-color: var(--mdc-theme-primary);
	/* Using your primary theme color */
	color: #ffffff;
	padding: 8px 16px;
	border-radius: 4px;
	text-decoration: none;
	cursor: pointer;
	transition: background-color 0.3s;
}

.directions-link:hover {
	background-color: #1565C0;
	/* Darkened primary color */
	opacity: 1;
	/* Ensure full opacity */
}

.start-driving-button {
	background-color: var(--mdc-theme-primary);
	color: #ffffff;
	border: none;
	padding: 8px 16px;
	border-radius: 4px;
	cursor: pointer;
	margin: 10px;
	font-size: 16px;
	transition: background-color 0.3s;
}

.start-driving-button:hover {
	background-color: #1565C0;
	/* Darkened primary color */
	opacity: 1;
	/* Ensure full opacity */
}

.tooltip {
	position: absolute;
	bottom: 10%;
	left: 50%;
	transform: translateX(-50%);
	padding: 8px 16px;
	background-color: rgba(0, 0, 0, 0.6);
	color: #fff;
	border-radius: 5px;
	z-index: 1;
	pointer-events: none;
	/* Ensure it doesn't interfere with map interactions */
	opacity: 0;
	/* Initially hidden */
	transition: opacity 0.3s ease;
	/* Smooth fade effect */
}
</style>
