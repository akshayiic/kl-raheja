<script>
	import bowser from 'bowser';
	import { onMount, onDestroy } from 'svelte';
	import { getContext } from 'svelte';
	import { writable, derived } from 'svelte/store';
	let Marzipano;

	const hotspotName = getContext('hotspotName');
	const currentUI = getContext('currentUI');
	$currentUI['overview'] = true;
	const UIPanel = getContext('UIPanel');

	// Available floors for each time of day
	const floors = [
		{ id: '0-2nd-floor', name: '2nd Floor' },
		{ id: '1-4th-floor', name: '4th Floor' },
		{ id: '2-6th-floor', name: '6th Floor' },
		{ id: '3-8th-floor', name: '8th Floor' },
		{ id: '4-10th-floor', name: '10th Floor' },
		{ id: '5-12th-floor', name: '12th Floor' },
		{ id: '6-14th-floor', name: '14th Floor' },
		{ id: '7-16th-floor', name: '16th Floor' },
		{ id: '8-18th-floor', name: '18th Floor' },
		{ id: '9-20th-floor', name: '20th Floor' },
		{ id: '10-22nd-floor', name: '22nd Floor' }
	];

	// Info hotspots data - using correct yaw/pitch from APP_DATA
	const infoHotspots = [
		{ yaw: 1.8844940263482037, pitch: -0.05054815947613989, title: 'Bandra Worli Sea Link' },
		{ yaw: -1.9542776260578663, pitch: 0.22421612137353364, title: 'Bandstand & Carter Road Promenade' },
		{ yaw: -0.4555211228580287, pitch: 0.01301193885635854, title: 'BKC (Bandra-Kurla Complex)' },
		{ yaw: -0.6126043828326306, pitch: 0.12261471785871336, title: 'Lilavati Hospital' },
		{ yaw: -0.28810601011943326, pitch: 0.059821744772238006, title: 'Dhirubhai Ambani International School' },
		{ yaw: -1.6807002432938933, pitch: 0.08104243591002991, title: 'Otters Club' },
		{ yaw: 2.073285248111959, pitch: -0.015141630319693178, title: 'Taj Lands End' },
		{ yaw: -0.1492642036394205, pitch: 0.02223561794271589, title: 'Jio World Centre (NMACC)' },
		{ yaw: -0.08302367651812048, pitch: 0.06513596391294918, title: 'Jio World Drive' },
		{ yaw: -2.1649118474316467, pitch: -0.010187657007763917, title: 'Versova–Bandra Sea Link' },
		{ yaw: 1.407461676916748, pitch: -0.06262553263736237, title: 'Worli business district' },
		{ yaw: 1.41569761010296, pitch: 0.22940729869127985, title: 'Mount Mary Basilica' },
		{ yaw: -0.32055851759378484, pitch: 0.06708659007874473, title: 'American School of Bombay' },
		{ yaw: -0.6243483273298018, pitch: 0.03932089879253553, title: 'Asian Heart Institute' },
		{ yaw: -1.1134199121902082, pitch: 0.013543448224595522, title: 'Pali Hill/Linking Road' }
	];

	// Initial view parameters from APP_DATA
	const initialView = {
		yaw: -2.9963666940833633,
		pitch: -0.029559784263739175,
		fov: 1.4438271227947086
	};

	// Scene levels/tile configuration
	const levels = [
		{ tileSize: 256, size: 256, fallbackOnly: true },
		{ tileSize: 512, size: 512 },
		{ tileSize: 512, size: 1024 },
		{ tileSize: 512, size: 2048 },
		{ tileSize: 512, size: 4096 }
	];

	let viewer;
	let allScenes = {};
	let currentScene = null;
	let viewChangeHandler = null;

	// Main overview uses 22nd floor
	const mainFloorIndex = 10;

	// State
	const selectedTime = writable('morning');
	const selectedFloorIndex = writable(10);
	const isMainOverview = derived(selectedFloorIndex, ($index) => $index === mainFloorIndex);

	// Hotspot toggle state
	const hotspotsEnabled = writable(true);

	// Track which hotspots have been shown
	const shownHotspots = new Set();

	onMount(async () => {
		// @ts-expect-error - marzipano doesn't have type bindings
		Marzipano = await import('marzipano');

		// Create viewer with settings from APP_DATA
		viewer = new Marzipano.Viewer(document.getElementById('pano'), {
			controls: {
				mouseViewMode: 'drag'
			}
		});

		window.dragControlMethod = bowser.mobile
			? viewer.controls().method('touchView').instance
			: viewer.controls().method('mouseViewDrag').instance;

		// Create scenes for each time of day and floor combination
		availableTimes.forEach((time) => {
			floors.forEach((floor) => {
				createScene(time, floor.id);
			});
		});

		// Load main overview by default
		loadScene('morning', 10);
	});

	// Time of day states
	const availableTimes = ['morning', 'afternoon', 'evening', 'night'];

	const createScene = (time, sceneId) => {
		// Scene path: {time}/app-files/tiles/{sceneId}
		const baseURL = 'https://assets.vestate.io/kl-rahega/drone-shots';
		const scenePath = `${time}/app-files/tiles/${sceneId}`;

		let source = Marzipano.ImageUrlSource.fromString(
			`${baseURL}/${scenePath}/{z}/{f}/{y}/{x}.jpg`,
			{ cubeMapPreviewUrl: `${baseURL}/${scenePath}/preview.jpg` }
		);

		let geometry = new Marzipano.CubeGeometry(levels);
		let limiter = Marzipano.RectilinearView.limit.traditional(4096, (130 * Math.PI) / 180);
		let view = new Marzipano.RectilinearView(initialView, limiter);

		let scene = viewer.createScene({
			source: source,
			geometry: geometry,
			view: view,
			pinFirstLevel: true
		});

		const sceneKey = `${time}_${sceneId}`;
		allScenes[sceneKey] = { source, view, scene, time };
	};

	const loadScene = (time, floorIndex) => {
		const floor = floors[floorIndex];
		if (!floor) return;

		const sceneKey = `${time}_${floor.id}`;
		const sceneData = allScenes[sceneKey];
		if (!sceneData) return;

		const scene = sceneData.scene;

		// Clear existing hotspots
		const container = scene.hotspotContainer();
		const hotspots = container.listHotspots();
		hotspots.forEach((hotspot) => {
			container.destroyHotspot(hotspot);
		});

		// Clear shown hotspots tracking
		shownHotspots.clear();

		// Remove previous view change handler
		if (viewChangeHandler) {
			scene.view().removeEventListener('change', viewChangeHandler);
		}

		// Add info hotspots only for main overview (22nd floor) AND when enabled
		const shouldShowHotspots = floorIndex === mainFloorIndex && $hotspotsEnabled;

		if (shouldShowHotspots) {
			infoHotspots.forEach((hotspot, index) => {
				createInfoHotspot(scene, hotspot, index);
			});

			// Add viewport visibility handler
			viewChangeHandler = () => {
				checkHotspotVisibility(scene);
			};
			scene.view().addEventListener('change', viewChangeHandler);

			// Initial check after scene loads
			setTimeout(() => {
				checkHotspotVisibility(scene);
			}, 300);
		}

		scene.switchTo();
		currentScene = scene;

		// Update state
		selectedTime.set(time);
		selectedFloorIndex.set(floorIndex);
	};

	const checkHotspotVisibility = (scene) => {
		if (!scene || !scene.view()) return;

		const view = scene.view();
		const yaw = view.yaw();
		const pitch = view.pitch();
		const fov = view.fov();

		// Check each hotspot
		infoHotspots.forEach((hotspot, index) => {
			const wrapper = document.getElementById(`hotspot-wrapper-${index}`);
			if (!wrapper) return;

			// Calculate angular distance considering yaw wraparound
			let yawDiff = Math.abs(hotspot.yaw - yaw);
			if (yawDiff > Math.PI) {
				yawDiff = 2 * Math.PI - yawDiff;
			}

			const pitchDiff = Math.abs(hotspot.pitch - pitch);

			// Check if hotspot is in viewport
			const inViewport = yawDiff < fov * 0.7 && pitchDiff < fov * 0.6;

			if (inViewport) {
				wrapper.classList.add('visible');
				shownHotspots.add(index);
			} else {
				wrapper.classList.remove('visible');
				shownHotspots.delete(index);
			}
		});
	};

	const selectTimeOfDay = (time) => {
		if (time === 'morning') {
			hotspotsEnabled.set(true);
		} else {
			hotspotsEnabled.set(false);
		}
		loadScene(time, $selectedFloorIndex);
	};

	const selectFloor = (index) => {
		loadScene($selectedTime, index);
	};

	const toggleHotspots = () => {
		hotspotsEnabled.update(enabled => !enabled);
		loadScene($selectedTime, $selectedFloorIndex);
	};

	const createInfoHotspot = (scene, hotspotData, index) => {
		// Create hotspot similar to interiors style
		const wrapper = document.createElement('div');
		wrapper.classList.add('info-hotspot');
		wrapper.id = `hotspot-wrapper-${index}`;

		// Hotspot element with text label
		const hotspot = document.createElement('div');
		hotspot.classList.add('hotspot');
		hotspot.innerText = hotspotData.title;

		wrapper.appendChild(hotspot);

		// Create hotspot at correct yaw/pitch position
		scene.hotspotContainer().createHotspot(
			wrapper,
			{ yaw: hotspotData.yaw, pitch: hotspotData.pitch },
			{}
		);
	};

	function capitalize(str) {
		return str.charAt(0).toUpperCase() + str.slice(1);
	}

	onDestroy(() => {
		if (viewer) {
			viewer = null;
		}
	});
</script>

<div class="overview-container">
	<!-- Time of Day Selector -->
	<div class="time-selector">
		<div class="time-selector-header">
			<svg width="20" height="20" viewBox="0 0 20 20" fill="none" xmlns="http://www.w3.org/2000/svg">
				<path d="M10 2C5.58172 2 2 5.58172 2 10C2 14.4183 5.58172 18 10 18C14.4183 18 18 14.4183 18 10C18 5.58172 14.4183 2 10 2ZM10 16C6.68629 16 4 13.3137 4 10C4 6.68629 6.68629 4 10 4C13.3137 4 16 6.68629 16 10C16 13.3137 13.3137 16 10 16Z" fill="currentColor"/>
				<path d="M11 6H9V10H11V6Z" fill="currentColor"/>
				<path d="M11 10H9V14H11V10Z" fill="currentColor"/>
			</svg>
			<span>Time of Day</span>
		</div>
		<div class="time-buttons">
			{#each availableTimes as time}
				<button
					class="time-btn {$selectedTime === time ? 'active' : ''}"
					on:click={() => selectTimeOfDay(time)}
				>
					{capitalize(time)}
				</button>
			{/each}
		</div>
	</div>

	<!-- Floor Selector -->
	<div class="floor-selector">
		<div class="floor-selector-header">
			<svg width="20" height="20" viewBox="0 0 20 20" fill="none" xmlns="http://www.w3.org/2000/svg">
				<path d="M3 3L10 1L17 3V10C17 14.97 13.5 19 10 20C6.5 19 3 14.97 3 10V3Z" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/>
				<path d="M8 10L10 12L14 8" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/>
			</svg>
			<span>Select Floor</span>
		</div>
		<div class="floor-buttons">
			{#each floors as floor, index}
				<button
					class="floor-btn {$selectedFloorIndex === index ? 'active' : ''}"
					on:click={() => selectFloor(index)}
				>
					{floor.name}
				</button>
			{/each}
		</div>
	</div>

	<!-- Hotspot Toggle -->
	{#if $isMainOverview}
		<div class="hotspot-toggle">
			<span class="toggle-label">Info Hotspots</span>
			<button
				class="toggle-switch {$hotspotsEnabled ? 'on' : 'off'}"
				on:click={toggleHotspots}
				aria-label={$hotspotsEnabled ? 'Turn off hotspots' : 'Turn on hotspots'}
			>
				<div class="toggle-slider"></div>
			</button>
		</div>
	{/if}

 

	<!-- Marzipano Viewer -->
	<div id="pano" class="pano-viewer"></div>
</div>

<style>
	.overview-container {
		position: fixed;
		top: 0;
		left: 0;
		width: 100%;
		height: 100%;
		background: #000;
	}

	.pano-viewer {
		width: 100%;
		height: 100%;
	}

	/* Time of Day Selector */
	.time-selector {
		position: fixed;
		top: 100px;
		right: 20px;
		z-index: 1000;
		background: rgba(15, 76, 92, 0.9);
		border-radius: 12px;
		padding: 16px;
		min-width: 180px;
		backdrop-filter: blur(10px);
		box-shadow: 0 8px 32px rgba(0, 0, 0, 0.3);
		max-height: 50vh;
		overflow-y: auto;
	}

	.time-selector-header {
		display: flex;
		align-items: center;
		gap: 8px;
		color: white;
		font-weight: 600;
		margin-bottom: 12px;
		padding-bottom: 8px;
		border-bottom: 1px solid rgba(255, 255, 255, 0.2);
	}

	.time-buttons {
		display: flex;
		flex-direction: column;
		gap: 8px;
	}

	.time-btn {
		padding: 10px 16px;
		border: 1px solid rgba(255, 255, 255, 0.3);
		border-radius: 8px;
		background: rgba(255, 255, 255, 0.1);
		color: white;
		font-size: 14px;
		font-weight: 500;
		cursor: pointer;
		transition: all 0.3s ease;
		text-transform: capitalize;
	}

	.time-btn:hover {
		background: rgba(255, 255, 255, 0.2);
		border-color: rgba(255, 255, 255, 0.5);
	}

	.time-btn.active {
		background: rgba(255, 212, 0, 0.3);
		border-color: #FFD400;
		color: #FFD400;
	}

	/* Floor Selector */
	.floor-selector {
		position: fixed;
		top: 100px;
		left: 20px;
		z-index: 1000;
		background: rgba(15, 76, 92, 0.9);
		border-radius: 12px;
		padding: 16px;
		min-width: 160px;
		backdrop-filter: blur(10px);
		box-shadow: 0 8px 32px rgba(0, 0, 0, 0.3);
		max-height: 70vh;
		overflow-y: auto;
	}

	.floor-selector-header {
		display: flex;
		align-items: center;
		gap: 8px;
		color: white;
		font-weight: 600;
		margin-bottom: 12px;
		padding-bottom: 8px;
		border-bottom: 1px solid rgba(255, 255, 255, 0.2);
	}

	.floor-buttons {
		display: flex;
		flex-direction: column;
		gap: 6px;
	}

	.floor-btn {
		padding: 8px 12px;
		border: 1px solid rgba(255, 255, 255, 0.3);
		border-radius: 6px;
		background: rgba(255, 255, 255, 0.1);
		color: white;
		font-size: 13px;
		font-weight: 500;
		cursor: pointer;
		transition: all 0.3s ease;
		text-align: left;
	}

	.floor-btn:hover {
		background: rgba(255, 255, 255, 0.2);
		border-color: rgba(255, 255, 255, 0.5);
		transform: translateX(2px);
	}

	.floor-btn.active {
		background: rgba(255, 212, 0, 0.3);
		border-color: #FFD400;
		color: #FFD400;
	}

	/* Hotspot Toggle */
	.hotspot-toggle {
		position: fixed;
		top: 20px;
		left: 50%;
		transform: translateX(-50%);
		z-index: 1000;
		background: rgba(15, 76, 92, 0.9);
		border-radius: 24px;
		padding: 10px 20px;
		display: flex;
		align-items: center;
		gap: 12px;
		backdrop-filter: blur(10px);
		box-shadow: 0 4px 16px rgba(0, 0, 0, 0.3);
	}

	.toggle-label {
		color: white;
		font-size: 14px;
		font-weight: 500;
	}

	.toggle-switch {
		position: relative;
		width: 48px;
		height: 24px;
		background: rgba(255, 255, 255, 0.2);
		border-radius: 12px;
		cursor: pointer;
		transition: background 0.3s ease;
		border: none;
		padding: 0;
	}

	.toggle-switch.on {
		background: #FFD400;
	}

	.toggle-slider {
		position: absolute;
		top: 2px;
		left: 2px;
		width: 20px;
		height: 20px;
		background: white;
		border-radius: 50%;
		transition: transform 0.3s ease;
		box-shadow: 0 2px 4px rgba(0, 0, 0, 0.2);
	}

	.toggle-switch.on .toggle-slider {
		transform: translateX(24px);
	}

	/* Overview Badge */
	.overview-badge {
		position: fixed;
		bottom: 20px;
		left: 50%;
		transform: translateX(-50%);
		z-index: 1000;
		background: rgba(255, 212, 0, 0.9);
		border-radius: 20px;
		padding: 10px 20px;
		display: flex;
		align-items: center;
		gap: 8px;
		color: #0F4C5C;
		font-size: 13px;
		font-weight: 600;
		box-shadow: 0 4px 16px rgba(255, 212, 0, 0.3);
		animation: slideUp 0.5s ease;
	}

	@keyframes slideUp {
		from {
			opacity: 0;
			transform: translateX(-50%) translateY(20px);
		}
		to {
			opacity: 1;
			transform: translateX(-50%) translateY(0);
		}
	}

	/* Info Hotspots - uses global .hotspot styles from styles.css */
	.info-hotspot {
		position: absolute;
	}

	/* Responsive */
	@media (max-width: 768px) {
		.time-selector {
			top: auto;
			bottom: 20px;
			right: 20px;
			min-width: 150px;
			max-height: 40vh;
		}

		.floor-selector {
			top: auto;
			bottom: 80px;
			left: 20px;
			min-width: 140px;
			max-height: 35vh;
		}

		.hotspot-toggle {
			top: 15px;
			padding: 8px 16px;
		}

		.toggle-label {
			font-size: 12px;
		}

		.overview-badge {
			bottom: auto;
			top: 65px;
		}

		.floor-btn {
			font-size: 12px;
			padding: 6px 10px;
		}
	}

	/* Scrollbar styling */
	.time-selector::-webkit-scrollbar,
	.floor-selector::-webkit-scrollbar {
		width: 6px;
	}

	.time-selector::-webkit-scrollbar-track,
	.floor-selector::-webkit-scrollbar-track {
		background: rgba(255, 255, 255, 0.1);
		border-radius: 3px;
	}

	.time-selector::-webkit-scrollbar-thumb,
	.floor-selector::-webkit-scrollbar-thumb {
		background: rgba(255, 255, 255, 0.3);
		border-radius: 3px;
	}

	.time-selector::-webkit-scrollbar-thumb:hover,
	.floor-selector::-webkit-scrollbar-thumb:hover {
		background: rgba(255, 255, 255, 0.5);
	}
</style>
