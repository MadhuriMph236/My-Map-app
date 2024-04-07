<script>
	import { onMount, afterUpdate } from "svelte";
	import L from "leaflet";
	import data from "./paris.json";
	let speedLimit = 50;
	let mymap;
	let geojsonLayer;
	let selectedRoad = null;
	let stats = {};
	let chart;

	onMount(() => {
		mymap = L.map("mapid").setView([48.8534, 2.3488], 13);
		L.tileLayer("https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png", {
			maxZoom: 19,
		}).addTo(mymap);
		calculateStats();
		updateMap();
	});

	// afterUpdate(() => {
	// 	if (chart) {
	// 		chart.destroy();
	// 	}

	// 	let ctx = document.getElementById("chart").getContext("2d");
	// 	chart = new Chart(ctx, {
	// 		type: "bar",
	// 		data: {
	// 			labels: Object.keys(stats.speedLimits),
	// 			datasets: [
	// 				{
	// 					label: "Number of Roads",
	// 					data: Object.values(stats.speedLimits),
	// 					backgroundColor: "rgba(75, 192, 192, 0.2)",
	// 					borderColor: "rgba(75, 192, 192, 1)",
	// 					borderWidth: 1,
	// 				},
	// 			],
	// 		},
	// 		options: {
	// 			scales: {
	// 				y: {
	// 					beginAtZero: true,
	// 				},
	// 			},
	// 		},
	// 	});
	// });
	function updateMap() {
		// Remove the old layer if it exists
		if (geojsonLayer) {
			mymap.removeLayer(geojsonLayer);
		}

		// Add the data to the map, filtering based on the speed limit
		geojsonLayer = L.geoJSON(data, {
			filter: (feature) => feature.properties.speedLimit <= speedLimit,
			onEachFeature: (feature, layer) => {
				// Add this function
				layer.on("click", () => {
					if (selectedRoad) {
						selectedRoad.setStyle({ color: "#3388ff" }); // Reset the color of the previously selected road
					}
					layer.setStyle({ color: "#ff0000" }); // Change the color of the selected road
					selectedRoad = layer;
					layer.bindTooltip(
						`Max Speed: ${feature.properties.speedLimit} km/h`,
					); // Bind a tooltip to the layer
				});
			},
		}).addTo(mymap);
	}
	function calculateStats() {
		let totalSpeedLimit = 0;
		let maxSpeedLimit = 0;
		let numRoads = data.features.length;

		data.features.forEach((feature) => {
			let speed = feature.properties.speedLimit;
			totalSpeedLimit += speed;
			if (speed > maxSpeedLimit) {
				maxSpeedLimit = speed;
			}
		});

		let avgSpeedLimit = totalSpeedLimit / numRoads;

		stats = {
			numRoads,
			avgSpeedLimit,
			maxSpeedLimit,
		};
	}
</script>

<div>
	<label for="speedLimit">Speed Limit:</label>
	<input
		id="speedLimit"
		type="number"
		bind:value={speedLimit}
		on:input={updateMap}
	/>
</div>
<div id="mapid" style="height: 300px;"></div>
{#if stats}
	<!-- Display the statistics -->
	<div>
		<h2>Road Network Statistics:</h2>
		<p>Number of Roads: {stats?.numRoads}</p>
		<p>Average Speed Limit: {stats?.avgSpeedLimit?.toFixed(2)} km/h</p>
		<p>Maximum Speed Limit: {stats?.maxSpeedLimit} km/h</p>
	</div>
{/if}
<canvas id="chart"></canvas>
