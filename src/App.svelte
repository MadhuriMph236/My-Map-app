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
		const shapefile = require("shapefile");
		mymap = L.map("mapid").setView([48.8534, 2.3488], 13);
		L.tileLayer("https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png", {
			maxZoom: 19,
		}).addTo(mymap);
		shapefile
			.open("./gis_osm_places_a_free_1.shp")
			.then((source) =>
				source.read().then(function log(result) {
					debugger;
					if (result.done) return;
					console.log(result.value);
					return source.read().then(log);
				}),
			)
			.catch((error) => console.error(error.stack));
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

<header class="header">
	<img style="width:250px" src="/logo.png" alt="Logo" id="logo" />
	<h1 id="username">User Name</h1>
</header>

<div class="test">
	<div class="center-input">
		<label for="speedLimit">Speed Limit:</label>
		<input
			id="speedLimit"
			type="number"
			bind:value={speedLimit}
			on:input={updateMap}
		/>
	</div>

	{#if stats}
		<div class="dashboard">
			<div class="card">
				<h2>Number of Roads</h2>
				<p>{stats?.numRoads}</p>
			</div>
			<div class="card">
				<h2>Average Speed Limit</h2>
				<p>{stats?.avgSpeedLimit?.toFixed(2)} km/h</p>
			</div>
			<div class="card">
				<h2>Maximum Speed Limit</h2>
				<p>{stats?.maxSpeedLimit} km/h</p>
			</div>
		</div>
	{/if}
	<div id="mapid" style="height: 450px;"></div>

	<canvas id="chart"></canvas>
</div>
<footer></footer>

<style>
	.header {
		display: flex;
		justify-content: space-between;
		align-items: center;
		padding: 1rem;
		background-color: #333;
		color: white;
	}
	.center-input {
		display: flex;
		justify-content: center;
		align-items: center;
	}
	.test {
		color: #fff;
	}
	#speedLimit {
		width: 40%;
		height: 40px;
		padding: 10px;
		border: none;
		border-radius: 20px;
		box-shadow: 0 0 15px 4px rgba(0, 0, 0, 0.06);
		background: #f5f5f5;
		outline: none;
		font-size: 16px;
		color: #555;
	}

	label {
		font-size: 18px;
		color: #555;
	}
	.dashboard {
		display: flex;
		justify-content: space-around;
		padding: 1em;
		background-color: #f5f5f5;
	}

	.card {
		flex: 1;
		box-shadow: 0 4px 8px 0 rgba(0, 0, 0, 0.2);
		padding: 1em;
		margin: 1em;
		background-color: #fff;
		text-align: center;
	}

	.card h2 {
		color: #333;
	}

	.card p {
		font-size: 2em;
		color: #666;
	}
</style>
