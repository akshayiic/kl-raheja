<script>
	import bowser from 'bowser';
	import { onMount, onDestroy } from 'svelte';
	import { getContext } from 'svelte';
	import { writable, derived } from 'svelte/store';
    import { goto } from '$app/navigation';
	let Marzipano;

	const hotspotName = getContext('hotspotName');
	const currentUI = getContext('currentUI');
	$currentUI['views'] = true;
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

	const reversedFloorsWithIndices = floors.map((f, i) => ({ ...f, originalIndex: i })).reverse();

	// Info hotspots data - using correct yaw/pitch from APP_DATA
	const infoHotspots = [
		{ yaw: 1.8844940263482037, pitch: -0.05054815947613989, title: 'Bandra Worli Sea Link', side: 'left', height: 80 },
		{
			yaw: -1.9542776260578663,
			pitch: 0.22421612137353364,
			title: 'Bandstand & Carter Road Promenade',
			side: 'right',
			height: 100
		},
		{ yaw: -0.4555211228580287, pitch: 0.01301193885635854, title: 'BKC (Bandra-Kurla Complex)', side: 'right', height: 140 },
		{ yaw: -0.6126043828326306, pitch: 0.12261471785871336, title: 'Lilavati Hospital', side: 'right', height: 170 },
		{
			yaw: -0.58810601011943326,
			pitch: 0.059821744772238006,
			title: 'Dhirubhai Ambani International School',
			side: 'left',
			height: 120
		},
		{ yaw: -1.6807002432938933, pitch: 0.08104243591002991, title: 'Otters Club', side: 'left', height: 140 },
		{ yaw: 2.073285248111959, pitch: -0.015141630319693178, title: 'Taj Lands End', side: 'right', height: 120 },
		{ yaw: -0.1492642036394205, pitch: 0.02223561794271589, title: 'Jio World Centre (NMACC)', side: 'left', height: 200 },
		{ yaw: -0.18302367651812048, pitch: 0.06513596391294918, title: 'Jio World Drive', side: 'right', height: 130 },
		{ yaw: -2.1649118474316467, pitch: -0.010187657007763917, title: 'Versova–Bandra Sea Link', side: 'left', height: 60 },
		{ yaw: 1.407461676916748, pitch: -0.06262553263736237, title: 'Worli business district', side: 'left', height: 90 },
		{ yaw: 1.41569761010296, pitch: 0.22940729869127985, title: 'Mount Mary Basilica', side: 'right', height: 150 },
		{ yaw: -0.32055851759378484, pitch: 0.06708659007874473, title: 'American School of Bombay', side: 'left', height: 65 },
		{ yaw: -0.6243483273298018, pitch: 0.03932089879253553, title: 'Asian Heart Institute', side: 'left', height: 60 },
		{ yaw: -1.1134199121902082, pitch: 0.013543448224595522, title: 'Pali Hill/Linking Road', side: 'left', height: 150 }
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

	// Collapsible sidebars state
	let floorCollapsed = true;
	let timeCollapsed = true;

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

		// Load main overview by default
		loadScene('morning', 10);
		UIPanel.set('loaded');
	});

	// Time of day states
	const availableTimes = ['morning', 'afternoon', 'evening', 'night'];

	const createScene = (time, sceneId) => {
		// Scene path: {time}/app-files/tiles/{sceneId}
		const baseURL = 'https://assets.vestate.io/kl-rahega/new-droneshots';

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

	// Track scene caching to prevent GPU memory starvation
	const MAX_CACHED_SCENES = 6;
	let sceneOrder = [];

	const loadScene = (time, floorIndex) => {
		const floor = floors[floorIndex];
		if (!floor) return;

		const sceneKey = `${time}_${floor.id}`;

		// Lazily create scene if it doesn't exist in cache yet
		if (!allScenes[sceneKey]) {
			createScene(time, floor.id);
		}

		// Move current scene to most-recently-used in cache order
		sceneOrder = sceneOrder.filter((key) => key !== sceneKey);
		sceneOrder.push(sceneKey);

		// Evict least-recently-used scenes if we exceed the cache cap
		while (sceneOrder.length > MAX_CACHED_SCENES) {
			const evictKey = sceneOrder.shift();
			if (evictKey && evictKey !== sceneKey && allScenes[evictKey]) {
				const evictData = allScenes[evictKey];
				viewer.destroyScene(evictData.scene);
				delete allScenes[evictKey];
			}
		}

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
		if (time !== 'morning') {
			hotspotsEnabled.set(false);
		}
		loadScene(time, $selectedFloorIndex);
	};

	const selectFloor = (index) => {
		loadScene($selectedTime, index);
	};

	const toggleHotspots = () => {
		hotspotsEnabled.update((enabled) => !enabled);
		loadScene($selectedTime, $selectedFloorIndex);
	};

	const hotspotImageMap = {
		'Bandra Worli Sea Link': {
			cat: 'Connectivity',
			folder: 'Bandra Worli Sea Link'
		},
		'Bandstand & Carter Road Promenade': {
			cat: 'Retail%20%26%20Lifestyle',
			folder: 'Bandstand'
		},
		'BKC (Bandra-Kurla Complex)': {
			cat: 'Commercial',
			folder: 'BKC'
		},
		'Lilavati Hospital': {
			cat: 'Hospitals',
			folder: 'Lilavati Hospital'
		},
		'Dhirubhai Ambani International School': {
			cat: 'Education%20Institutes',
			folder: 'Dhirubhai Ambani International School'
		},
		'Otters Club': {
			cat: 'Cafe%20%26%20Clubs',
			folder: 'Otter_s'
		},
		'Taj Lands End': {
			cat: 'Cafe%20%26%20Clubs',
			folder: 'Taj Lounges'
		},
		'Jio World Centre (NMACC)': {
			cat: 'Commercial',
			folder: 'Jio World Centre'
		},
		'Jio World Drive': {
			cat: 'Retail%20%26%20Lifestyle',
			folder: 'Jio World Drive'
		},
		'Versova–Bandra Sea Link': {
			cat: 'Connectivity',
			folder: 'Versova-Bandra sea link'
		},
		'Worli business district': {
			cat: 'Commercial',
			folder: 'Worli'
		},
		'Mount Mary Basilica': {
			cat: 'Faith%20%26%20Heritage',
			folder: 'Mount Mary'
		},
		'American School of Bombay': {
			cat: 'Education%20Institutes',
			folder: 'American School of Bombay'
		},
		'Asian Heart Institute': {
			cat: 'Hospitals',
			folder: 'Asian Heart Institute'
		},
		'Pali Hill/Linking Road': {
			cat: 'Retail%20%26%20Lifestyle',
			folder: 'Linking-Hill Road'
		}
	};

	const createInfoHotspot = (scene, hotspotData, index) => {
		const wrapper = document.createElement('div');
		wrapper.classList.add('info-hotspot');
		wrapper.classList.add('overview-hotspot');
		
		// Use hand-tailored side alignment from data (fallback to alternating left/right)
		const side = hotspotData.side || (index % 2 === 0 ? 'right' : 'left');
		wrapper.classList.add(`hotspot-${side}`);
		
		wrapper.id = `hotspot-wrapper-${index}`;

		// Use hand-tailored height from data (fallback to 75)
		const height = hotspotData.height || 75;
		wrapper.style.setProperty('--hotspot-height', `${height}px`);

		// Center pulse dot
		const pulseDot = document.createElement('div');
		pulseDot.classList.add('hotspot-pulse-dot');
		wrapper.appendChild(pulseDot);

		// Vertical line
		const line = document.createElement('div');
		line.classList.add('hotspot-line');
		wrapper.appendChild(line);

		// Hotspot label
		const hotspot = document.createElement('div');
		hotspot.classList.add('hotspot');
		hotspot.classList.add('overview-hotspot-label');

		const imgData = hotspotImageMap[hotspotData.title];
		if (imgData) {
			const img = document.createElement('img');
			img.src = `https://assets.vestate.io/kl-rahega/images/${imgData.cat}/${encodeURIComponent(imgData.folder)}/1.png`;
			img.alt = hotspotData.title;
			img.classList.add('hotspot-label-img');
			hotspot.appendChild(img);
		}

		const textContainer = document.createElement('div');
		textContainer.classList.add('hotspot-label-text-container');

		const titleEl = document.createElement('div');
		titleEl.classList.add('hotspot-label-title');
		titleEl.innerText = hotspotData.title;
		textContainer.appendChild(titleEl);

		const subtitleEl = document.createElement('div');
		subtitleEl.classList.add('hotspot-label-subtitle');
		subtitleEl.innerText = 'Mumbai';
		textContainer.appendChild(subtitleEl);

		hotspot.appendChild(textContainer);
		wrapper.appendChild(hotspot);

		// Create hotspot at correct yaw/pitch position
		scene
			.hotspotContainer()
			.createHotspot(wrapper, { yaw: hotspotData.yaw, pitch: hotspotData.pitch }, {});
	};

	function capitalize(str) {
		return str.charAt(0).toUpperCase() + str.slice(1);
	}

	let lastFloorWheelTime = 0;
	let lastTimeWheelTime = 0;

	function handleFloorWheel(e) {
		e.preventDefault();
		const now = Date.now();
		if (now - lastFloorWheelTime < 250) {
			return;
		}
		lastFloorWheelTime = now;

		// Scroll up (deltaY < 0) goes to higher floor index, scroll down (deltaY > 0) goes to lower floor index
		const direction = e.deltaY < 0 ? 1 : -1;
		const nextIndex = $selectedFloorIndex + direction;
		if (nextIndex >= 0 && nextIndex < floors.length) {
			selectFloor(nextIndex);
		}
	}

	function handleTimeWheel(e) {
		e.preventDefault();
		const now = Date.now();
		if (now - lastTimeWheelTime < 250) {
			return;
		}
		lastTimeWheelTime = now;

		// Scroll down (deltaY > 0) goes to later time of day, scroll up (deltaY < 0) goes to earlier time of day
		const direction = e.deltaY > 0 ? 1 : -1;
		const activeTimeIdx = availableTimes.indexOf($selectedTime);
		const nextIndex = activeTimeIdx + direction;
		if (nextIndex >= 0 && nextIndex < availableTimes.length) {
			selectTimeOfDay(availableTimes[nextIndex]);
		}
	}

	let floorTouchStartY = 0;
	let timeTouchStartY = 0;
	const TOUCH_SWIPE_THRESHOLD = 30;

	function handleFloorTouchStart(e) {
		floorTouchStartY = e.touches[0].clientY;
	}

	function handleFloorTouchMove(e) {
		e.preventDefault();
		const currentY = e.touches[0].clientY;
		const diffY = floorTouchStartY - currentY; // positive = swipe up

		if (Math.abs(diffY) >= TOUCH_SWIPE_THRESHOLD) {
			const direction = diffY > 0 ? 1 : -1;
			const nextIndex = $selectedFloorIndex + direction;
			if (nextIndex >= 0 && nextIndex < floors.length) {
				selectFloor(nextIndex);
			}
			floorTouchStartY = currentY;
		}
	}

	function handleTimeTouchStart(e) {
		timeTouchStartY = e.touches[0].clientY;
	}

	function handleTimeTouchMove(e) {
		e.preventDefault();
		const currentY = e.touches[0].clientY;
		const diffY = timeTouchStartY - currentY; // positive = swipe up

		if (Math.abs(diffY) >= TOUCH_SWIPE_THRESHOLD) {
			const direction = diffY > 0 ? 1 : -1;
			const activeTimeIdx = availableTimes.indexOf($selectedTime);
			const nextIndex = activeTimeIdx + direction;
			if (nextIndex >= 0 && nextIndex < availableTimes.length) {
				selectTimeOfDay(availableTimes[nextIndex]);
			}
			timeTouchStartY = currentY;
		}
	}

	onDestroy(() => {
		if (viewer) {
			viewer = null;
		}
	});
</script>

<div class="overview-container">
	{#if timeCollapsed}
		<button
			class="minimized-sidebar-tab-right"
			on:click={() => (timeCollapsed = false)}
			type="button"
			id="minimize-toggle-time"
		>
			<span class="chevron-group">
				<svg width="24" height="24" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
					<path class="chevron-arrow-1" d="M17 18L11 12L17 6" stroke="rgba(255, 255, 255, 0.85)" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round"/>
					<path class="chevron-arrow-2" d="M10 18L4 12L10 6" stroke="rgba(255, 255, 255, 0.35)" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round"/>
				</svg>
			</span>
			<span class="golden-circle">
				<img src="/timearrow.png" alt="Time Arrow" class="w-5 h-5 object-contain" />
			</span>
		</button>
	{/if}

	<!-- Time of Day Selector (Semi-circular dial on right edge) -->
	<div 
		class="time-dial {timeCollapsed ? 'collapsed' : ''}" 
		on:wheel|preventDefault|stopPropagation|nonpassive={handleTimeWheel}
		on:touchstart|nonpassive={handleTimeTouchStart}
		on:touchmove|preventDefault|stopPropagation|nonpassive={handleTimeTouchMove}
	>
		<!-- Collapse/Expand Arrow -->
		{#if !timeCollapsed}
			<button
				class="dial-toggle-btn right-dial-toggle"
				on:click={() => (timeCollapsed = !timeCollapsed)}
				type="button"
			>
				<svg
					class="w-4 h-4 text-white transition-transform duration-300"
					viewBox="0 0 24 24"
					fill="none"
					stroke="currentColor"
					stroke-width="2.5"
					stroke-linecap="round"
					stroke-linejoin="round"
				>
					<polyline points="9 18 15 12 9 6"></polyline>
				</svg>
			</button>
		{/if}

		<!-- Center text labels for Time Dial -->
		<div class="dial-center-label right-dial-label">
			<span class="text-[8px] tracking-widest text-white/40 uppercase">Sun Angle</span>
			<span class="text-[19px] font-semibold text-white mt-1 uppercase" style="font-family: 'The Seasons', serif;">
				{$selectedTime}
			</span>
		</div>

		<!-- Radial Tick Marks -->
		{#each Array(20) as _, i}
			{@const tickAngle = -60 + (i * 120) / 19}
			{@const tickRad = (tickAngle * Math.PI) / 180}
			{@const tx = Math.cos(tickRad) * 175}
			{@const ty = Math.sin(tickRad) * 175}
			<div 
				class="dial-tick" 
				style="right: {tx}px; top: calc(50% + {ty}px); transform: translate(50%, -50%) rotate({-tickAngle}deg);"
			></div>
		{/each}

		<!-- Time Buttons along the arc -->
		<div class="dial-buttons">
			{#each availableTimes as time, idx}
				{@const activeTimeIdx = availableTimes.indexOf($selectedTime)}
				{@const relativeDistance = idx - activeTimeIdx}
				{@const angle = relativeDistance * 30}
				{@const rad = (angle * Math.PI) / 180}
				{@const tx = Math.cos(rad) * 185}
				{@const ty = Math.sin(rad) * 185}
				{@const opacity = Math.max(0, 1 - Math.abs(angle) / 75)}
				{@const visible = Math.abs(angle) <= 75}
				<button
					class="dial-time-btn {$selectedTime === time ? 'active' : ''}"
					style="right: {tx}px; top: calc(50% + {ty}px); opacity: {opacity}; pointer-events: {visible ? 'auto' : 'none'};"
					on:click={() => selectTimeOfDay(time)}
					title={capitalize(time)}
				>
					{#if time === 'morning'}
						<svg class="{$selectedTime === time ? 'w-5 h-5' : 'w-4 h-4'}" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
							<circle cx="12" cy="12" r="4"/>
							<path d="M12 2v2M12 20v2M4.93 4.93l1.41 1.41M17.66 17.66l1.41 1.41M2 12h2M20 12h2M6.34 17.66l-1.41 1.41M19.07 4.93l-1.41 1.41"/>
						</svg>
					{:else if time === 'afternoon'}
						<svg class="{$selectedTime === time ? 'w-5 h-5' : 'w-4 h-4'}" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
							<circle cx="12" cy="12" r="5"/>
							<path d="M12 1v2M12 21v2M4.22 4.22l1.42 1.42M18.36 18.36l1.42 1.42M1 12h2M21 12h2M5.64 18.36l-1.42 1.42M19.78 4.22l-1.42 1.42"/>
						</svg>
					{:else if time === 'evening'}
						<svg class="{$selectedTime === time ? 'w-5 h-5' : 'w-4 h-4'}" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
							<path d="M17 18a5 5 0 0 0-10 0M12 2v7M4.93 4.93l4.24 4.24M19.07 4.93l-4.24 4.24M2 18h20"/>
						</svg>
					{:else}
						<svg class="{$selectedTime === time ? 'w-5 h-5' : 'w-4 h-4'}" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
							<path d="M12 3a6 6 0 0 0 9 9 9 9 0 1 1-9-9Z"/>
						</svg>
					{/if}
				</button>
			{/each}
		</div>
	</div>

	{#if floorCollapsed}
		<button
			class="minimized-sidebar-tab"
			style="position: fixed; left: 0; top: 50%; transform: translateY(-50%); z-index: 1001; margin-top: 0;"
			on:click={() => (floorCollapsed = false)}
			type="button"
			id="minimize-toggle-floor"
		>
			<span class="chevron-group">
				<svg width="24" height="24" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
					<path class="chevron-arrow-1" d="M7 6L13 12L7 18" stroke="rgba(255, 255, 255, 0.35)" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round"/>
					<path class="chevron-arrow-2" d="M14 6L20 12L14 18" stroke="rgba(255, 255, 255, 0.85)" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round"/>
				</svg>
			</span>
			<span class="golden-circle">
				<svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="white" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
					<polygon points="12 2 2 7 12 12 22 7 12 2"></polygon>
					<polyline points="2 17 12 22 22 17"></polyline>
					<polyline points="2 12 12 17 22 12"></polyline>
				</svg>
			</span>
		</button>
	{/if}

	<!-- Floor Selector (Semi-circular dial on left edge) -->
	<div 
		class="floor-dial {floorCollapsed ? 'collapsed' : ''}" 
		on:wheel|preventDefault|stopPropagation|nonpassive={handleFloorWheel}
		on:touchstart|nonpassive={handleFloorTouchStart}
		on:touchmove|preventDefault|stopPropagation|nonpassive={handleFloorTouchMove}
	>
		<!-- Collapse/Expand Arrow -->
		{#if !floorCollapsed}
			<button
				class="dial-toggle-btn left-dial-toggle"
				on:click={() => (floorCollapsed = !floorCollapsed)}
				type="button"
			>
				<svg
					class="w-4 h-4 text-white transition-transform duration-300"
					viewBox="0 0 24 24"
					fill="none"
					stroke="currentColor"
					stroke-width="2.5"
					stroke-linecap="round"
					stroke-linejoin="round"
				>
					<polyline points="15 18 9 12 15 6"></polyline>
				</svg>
			</button>
		{/if}

		<!-- Center text labels -->
		<div class="dial-center-label">
			<span class="text-[12px] tracking-widest text-white/40 uppercase">Total Floors</span>
			<span class="text-3xl font-semibold text-white mt-1 mx-auto"  >
				22<span class="text-xs font-normal align-top ml-0.5  ">th</span>
			</span>
		</div>

		<!-- Radial Tick Marks -->
		{#each Array(32) as _, i}
			{@const tickAngle = -72 + (i * 144) / 31}
			{@const tickRad = (tickAngle * Math.PI) / 180}
			{@const tx = Math.cos(tickRad) * 175}
			{@const ty = Math.sin(tickRad) * 175}
			<div 
				class="dial-tick" 
				style="left: {tx}px; top: calc(50% + {ty}px); transform: translate(-50%, -50%) rotate({tickAngle}deg);"
			></div>
		{/each}

		<!-- Floor Buttons along the arc -->
		<div class="dial-buttons">
			{#each reversedFloorsWithIndices as floor, idx}
				{@const activeFloorIdx = reversedFloorsWithIndices.findIndex(f => f.originalIndex === $selectedFloorIndex)}
				{@const relativeDistance = activeFloorIdx - idx}
				{@const angle = relativeDistance * 24}
				{@const rad = (angle * Math.PI) / 180}
				{@const fx = Math.cos(rad) * 175}
				{@const fy = Math.sin(rad) * 175}
				{@const opacity = Math.max(0, 1 - Math.abs(angle) / 75)}
				{@const visible = Math.abs(angle) <= 75}
				{@const floorNum = floor.name.match(/\d+/)[0]}
				<button
					class="dial-floor-btn {$selectedFloorIndex === floor.originalIndex ? 'active' : ''}"
					style="left: {fx}px; top: calc(50% + {fy}px); opacity: {opacity}; pointer-events: {visible ? 'auto' : 'none'};"
					on:click={() => selectFloor(floor.originalIndex)}
				>
					{floorNum}
				</button>
			{/each}
		</div>
	</div>

	<!-- Hotspot Toggle Pill -->
	{#if $isMainOverview && $selectedTime === 'morning'}
		<div class="hotspot-toggle-pill animate-fade-in">
			<span class="toggle-pill-label">Info Hotspots</span>
			<button
				class="toggle-pill-btn {$hotspotsEnabled ? 'active' : ''}"
				on:click={toggleHotspots}
				aria-label={$hotspotsEnabled ? 'Turn off hotspots' : 'Turn on hotspots'}
				type="button"
			>
				<svg class="w-3.5 h-3.5" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round">
					<circle cx="12" cy="12" r="10"></circle>
					<line x1="12" y1="16" x2="12" y2="12"></line>
					<line x1="12" y1="8" x2="12.01" y2="8"></line>
				</svg>
			</button>
		</div>
	{/if}

	<!-- Go Back Button at bottom-left corner -->
	<button
		class="go-back-btn animate-fade-in"
		on:click={() => {
			$currentUI.views = false;
			goto('/menu');
		}}
		type="button"
	>
		<svg class="w-3.5 h-3.5 mr-1.5" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round">
			<line x1="19" y1="12" x2="5" y2="12"></line>
			<polyline points="12 19 5 12 12 5"></polyline>
		</svg>
		Go Back
	</button>

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

	/* Semi-circular dials layout */
	.floor-dial {
		position: fixed;
		left: -20px;
		top: 50%;
		transform: translateY(-50%);
		width: 220px;
		height: 440px;
		border-radius: 0 220px 220px 0;
		background: rgba(18, 18, 18, 0.55);
		backdrop-filter: blur(20px);
		-webkit-backdrop-filter: blur(20px);
		border: 1px solid rgba(255, 255, 255, 0.08);
		border-left: none;
		box-shadow: 0 10px 40px rgba(0, 0, 0, 0.5);
		z-index: 1000;
		transition: transform 0.6s cubic-bezier(0.16, 1, 0.3, 1);
	}

	.floor-dial.collapsed {
		transform: translate(-200px, -50%);
	}

	.time-dial {
		position: fixed;
		right: -40px;
		top: 50%;
		transform: translateY(-50%);
		width: 230px;
		height: 440px;
		border-radius: 220px 0 0 220px;
		background: rgba(18, 18, 18, 0.55);
		backdrop-filter: blur(20px);
		-webkit-backdrop-filter: blur(20px);
		border: 1px solid rgba(255, 255, 255, 0.08);
		border-right: none;
		box-shadow: 0 10px 40px rgba(0, 0, 0, 0.5);
		z-index: 1000;
		transition: transform 0.6s cubic-bezier(0.16, 1, 0.3, 1);
	}

	.time-dial.collapsed {
		transform: translate(215px, -50%);
	}

	/* Toggle buttons styling */
	.dial-toggle-btn {
		position: absolute;
		width: 36px;
		height: 36px;
		border-radius: 50%;
		border: 1.2px solid rgba(255, 255, 255, 0.15);
		background: rgba(30, 30, 30, 0.45);
		backdrop-filter: blur(10px);
		display: flex;
		align-items: center;
		justify-content: center;
		cursor: pointer;
		transition: all 0.4s cubic-bezier(0.25, 1, 0.5, 1);
		z-index: 30;
		padding: 0;
	}

	.left-dial-toggle {
		left: 200px;
		top: 50%;
		transform: translateY(-50%);
	}

	.right-dial-toggle {
		right: 210px;
		top: 50%;
		transform: translateY(-50%);
	}

	.right-dial-toggle.active-golden {
		background: rgba(222, 173, 102, 1) !important;
		border-color: rgba(222, 173, 102, 1) !important;
		box-shadow: 0 0 15px rgba(222, 173, 102, 0.5);
		width: 48px;
		height: 48px;
		right: 145px;
	}

	/* Center labels for Floor Dial */
	.dial-center-label {
		position: absolute;
		left: 40px;
		top: 50%;
		transform: translateY(-50%);
		display: flex;
		flex-direction: column;
		align-items: flex-start;
		pointer-events: none;
		user-select: none;
	}

	.right-dial-label { 
		left: 70px !important;
		right: auto !important;
		align-items: flex-start !important;
		text-align: left !important;
	}

	/* Dial buttons wrapper and absolute layout */
	.dial-buttons {
		position: absolute;
		inset: 0;
		pointer-events: none;
	}

	/* Radial Ticks */
	.dial-tick {
		position: absolute;
		width: 6px;
		height: 1px;
		background: rgba(255, 255, 255, 0.18);
		transform-origin: left center;
		pointer-events: none;
	}

	/* Floor Button along arc */
	.dial-floor-btn {
		position: absolute;
		width: 28px;
		height: 28px;
		border-radius: 50%;
		border: none;
		background: transparent;
		color: rgba(255, 255, 255, 0.5);
		font-size: 20px;
		font-weight: 500;
		cursor: pointer;
		display: flex;
		align-items: center;
		justify-content: center;
		pointer-events: auto;
		transform: translate(-50%, -50%);
		transition: all 0.3s cubic-bezier(0.25, 1, 0.5, 1);
		padding: 0;
	}

	.dial-floor-btn:hover {
		color: white;
		font-weight: bold;
		scale: 1.15;
	}

	.dial-floor-btn.active {
		background: rgba(222, 173, 102, 1) !important;
		color: white !important;
		font-weight: bold;
		width: 48px;
		height: 35px;
		font-size: 20px;
		border-radius: 9999px !important;
		box-shadow: 0 0 15px rgba(222, 173, 102, 0.45);
	}

	/* Time Button along arc */
	.dial-time-btn {
		position: absolute;
		width: 36px;
		height: 36px;
		border-radius: 50%;
		border: 1px solid rgba(255, 255, 255, 0.15);
		background: rgba(0, 0, 0, 0.35);
		color: rgba(255, 255, 255, 0.6);
		cursor: pointer;
		display: flex;
		align-items: center;
		justify-content: center;
		pointer-events: auto;
		transform: translate(50%, -50%);
		transition: all 0.3s cubic-bezier(0.25, 1, 0.5, 1);
		padding: 0;
	}

	.dial-time-btn:hover {
		border-color: white;
		color: white;
		scale: 1.1;
	}

	.dial-time-btn.active {
		background: rgba(222, 173, 102, 1) !important;
		border-color: rgba(222, 173, 102, 1) !important;
		color: white !important;
		width: 48px;
		height: 48px;
		box-shadow: 0 0 15px rgba(222, 173, 102, 0.45);
	}
	
	/* Hotspot Toggle Pill at Top Center */
	.hotspot-toggle-pill {
		position: fixed;
		top: 20px;
		left: 50%;
		transform: translateX(-50%);
		z-index: 1000;
		background: rgba(30, 30, 30, 0.45);
		border-radius: 9999px;
		padding: 8px 12px 8px 20px;
		display: flex;
		align-items: center;
		gap: 16px;
		backdrop-filter: blur(10px);
		-webkit-backdrop-filter: blur(10px);
		border: 1.2px solid rgba(255, 255, 255, 0.15);
		box-shadow: 0 4px 20px rgba(0, 0, 0, 0.35);
	}

	.toggle-pill-label {
		color: rgba(255, 255, 255, 0.85);
		font-size: 11px;
		font-weight: 500;
		letter-spacing: 0.08em; 
		font-family: 'Imprima', sans-serif;
	}

	.toggle-pill-btn {
		width: 28px;
		height: 28px;
		border-radius: 50%;
		border: none;
		background: rgba(0, 0, 0, 0.45);
		color: rgba(255, 255, 255, 0.6);
		display: flex;
		align-items: center;
		justify-content: center;
		cursor: pointer;
		transition: all 0.3s cubic-bezier(0.25, 1, 0.5, 1);
		padding: 0;
	}

	.toggle-pill-btn:hover {
		color: white;
		background: rgba(0, 0, 0, 0.65);
	}

	.toggle-pill-btn.active {
		background: rgba(222, 173, 102, 1) !important;
		color: white !important;
		box-shadow: 0 0 10px rgba(222, 173, 102, 0.4);
	}

	/* Go Back Button */
	.go-back-btn {
		position: fixed;
		bottom: 24px;
		left: 24px;
		z-index: 1000;
		background: rgba(30, 30, 30, 0.45);
		border-radius: 9999px;
		padding: 8px 20px;
		display: flex;
		align-items: center;
		justify-content: center;
		backdrop-filter: blur(10px);
		-webkit-backdrop-filter: blur(10px);
		border: 1.2px solid rgba(255, 255, 255, 0.15);
		box-shadow: 0 4px 20px rgba(0, 0, 0, 0.35);
		color: rgba(255, 255, 255, 0.85);
		font-size: 11px;
		font-weight: 500;
		letter-spacing: 0.08em;
		text-transform: uppercase;
		cursor: pointer;
		transition: all 0.3s cubic-bezier(0.25, 1, 0.5, 1);
		font-family: 'Imprima', sans-serif;
	}

	.go-back-btn:hover {
		background: rgba(222, 173, 102, 1);
		border-color: rgba(222, 173, 102, 1);
		color: white;
		box-shadow: 0 0 15px rgba(222, 173, 102, 0.45);
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
		color: #0f5da8;
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

	/* Responsive styling for semi-circular dials and controls */
	@media (max-width: 768px), (max-width: 950px) and (orientation: landscape) {
		.floor-dial {
			width: 220px !important;
			transform: translateY(-50%) scale(0.65) !important;
			transform-origin: left center;
		}
		.floor-dial.collapsed {
			transform: translate(-225px, -50%) scale(0.65) !important;
			transform-origin: left center;
		}

		.time-dial {
			width: 250px !important;
			right: -10px !important;
			transform: translateY(-50%) scale(0.65) !important;
			transform-origin: right center;
		}
		.time-dial.collapsed {
			transform: translate(225px, -50%) scale(0.65) !important;
			transform-origin: right center;
		}
		.right-dial-toggle {
			right: 230px !important;
		}

		.dial-center-label:not(.right-dial-label) span:nth-child(2) {
			font-size: 14px !important;
		}

		.right-dial-label {
			right: auto !important;
			left: 110px !important;
			align-items: flex-start !important;
			text-align: left !important;
		}
		.right-dial-label span:nth-child(1) {
			font-size: 7px !important;
		}
		.right-dial-label span:nth-child(2) {
			font-size: 12px !important;
			letter-spacing: 0.05em !important;
		}

		:global(.minimized-sidebar-tab) {
			transform: translateY(-50%) scale(0.65) !important;
			transform-origin: left center !important;
			margin-top: 0 !important;
		}
		
		:global(.minimized-sidebar-tab-right) {
			transform: translateY(-50%) scale(0.65) !important;
			transform-origin: right center !important;
		}

		:global(.overview-hotspot .hotspot-pulse-dot) {
			width: 32px !important;
			height: 32px !important;
			left: -16px !important;
			top: -16px !important;
		}

		.go-back-btn {
			bottom: 16px !important;
			left: 16px !important;
			padding: 6px 16px !important;
			font-size: 10px !important;
		}

		.hotspot-toggle-pill {
			top: 16px !important;
			padding: 6px 12px 6px 16px !important;
			gap: 10px !important;
		}
	}

	@media (max-width: 480px) {
		.floor-dial {
			transform: translateY(-50%) scale(0.52) !important;
			transform-origin: left center;
		}
		.floor-dial.collapsed {
			transform: translate(-185px, -50%) scale(0.52) !important;
			transform-origin: left center;
		}

		.time-dial {
			right: 0px !important;
			transform: translateY(-50%) scale(0.52) !important;
			transform-origin: right center;
		}
		.time-dial.collapsed {
			transform: translate(220px, -50%) scale(0.52) !important;
			transform-origin: right center;
		}
		.right-dial-toggle {
			right: 230px !important;
		}
		.right-dial-label {
			left: 100px !important;
			right: auto !important;
		}

		:global(.minimized-sidebar-tab) {
			transform: translateY(-50%) scale(0.52) !important;
			transform-origin: left center !important;
			margin-top: 0 !important;
		}
		
		:global(.minimized-sidebar-tab-right) {
			transform: translateY(-50%) scale(0.52) !important;
			transform-origin: right center !important;
		}

		:global(.overview-hotspot .hotspot-pulse-dot) {
			width: 26px !important;
			height: 26px !important;
			left: -13px !important;
			top: -13px !important;
		}
	}

	/* Hotspots Animations & Glassmorphism styles */
	:global(.overview-hotspot) {
		position: absolute;
		pointer-events: none;
	}

	:global(.overview-hotspot .hotspot-pulse-dot) {
		position: absolute;
		width: 44px;
		height: 44px;
		background: url('/hotspot.png') no-repeat center center;
		background-size: contain;
		left: -22px;
		top: -22px;
		z-index: 2;
		transition: transform 0.25s cubic-bezier(0.25, 1, 0.5, 1);
	}

	:global(.overview-hotspot .hotspot-pulse-dot:hover) {
		transform: scale(1.15);
	}



	:global(.overview-hotspot .hotspot-line) {
		position: absolute;
		bottom: 0;
		left: -1px;
		width: 2px;
		height: var(--hotspot-height, 75px);
		background: linear-gradient(to top, #ffffff 0%, rgba(255, 255, 255, 0.8) 40%, rgba(255, 255, 255, 0.1) 100%);
		transform-origin: bottom;
		transform: scaleY(0);
	}
	
	:global(.overview-hotspot .hotspot-pulse-dot) {
		pointer-events: auto !important;
		cursor: pointer !important;
	}

	:global(.overview-hotspot.visible .hotspot-line) {
		transform: scaleY(0) !important;
	}
	:global(.overview-hotspot.visible:hover .hotspot-line) {
		transform: scaleY(1) !important;
	}

	:global(.overview-hotspot .overview-hotspot-label) {
		/* Reset global .hotspot styles */
		margin-top: 0 !important;
		padding: 6px 28px 6px 6px !important;
		border-radius: 9999px !important;
		
		/* White capsule background */
		background: #ffffff !important;
		border: 1.2px solid rgba(0, 0, 0, 0.08) !important;
		text-shadow: none !important;

		/* Flex layout */
		display: flex !important;
		align-items: center !important;
		gap: 16px !important;

		/* Positioning and initial state */
		position: absolute !important;
		bottom: var(--hotspot-height, 75px) !important;
		pointer-events: auto !important;
		cursor: pointer !important;
		white-space: nowrap !important;
		z-index: 3;
	}

	/* Image styling inside label */
	:global(.overview-hotspot .hotspot-label-img) {
		width: 60px !important;
		height: 60px !important;
		border-radius: 50% !important;
		object-fit: cover !important;
		display: block !important;
		pointer-events: none !important;
	}

	/* Text container styling */
	:global(.overview-hotspot .hotspot-label-text-container) {
		display: flex !important;
		flex-direction: column !important;
		align-items: flex-start !important;
		justify-content: center !important;
		gap: 2px !important;
		pointer-events: none !important;
	}

	/* Title styling */
	:global(.overview-hotspot .hotspot-label-title) {
		color: #b08d4f !important;
		font-size: 17px !important;
		font-weight: 550 !important;
		font-family: 'Outfit', 'Inter', sans-serif !important;
		line-height: 1.2 !important;
	}

	/* Subtitle styling */
	:global(.overview-hotspot .hotspot-label-subtitle) {
		color: #2c3539 !important;
		font-size: 14px !important;
		font-weight: 400 !important;
		font-family: 'Outfit', 'Inter', sans-serif !important;
		line-height: 1.2 !important;
	}

	/* Remove the ::before arrow from our overview hotspot */
	:global(.overview-hotspot .overview-hotspot-label::before) {
		display: none !important;
		content: none !important;
	}

	/* Alignment-specific styles */
	:global(.overview-hotspot.hotspot-right .overview-hotspot-label) {
		left: -15px !important;
		right: auto !important;
		transform: translateX(-15px) scale(0.95) !important;
		opacity: 0 !important;
		clip-path: inset(0 100% 0 0) !important;
		transition: opacity 0.5s cubic-bezier(0.25, 1, 0.5, 1), 
		            transform 0.5s cubic-bezier(0.25, 1, 0.5, 1),
		            clip-path 0.6s cubic-bezier(0.25, 1, 0.5, 1),
		            background-color 0.3s ease,
		            border-color 0.3s ease,
		            box-shadow 0.3s ease !important;
	}

	:global(.overview-hotspot.hotspot-right.visible:hover .overview-hotspot-label) {
		opacity: 1 !important;
		transform: translateX(0) scale(1) !important;
		clip-path: inset(0 0 0 0) !important;
	}

	:global(.overview-hotspot.hotspot-left .overview-hotspot-label) {
		right: -15px !important;
		left: auto !important;
		transform: translateX(15px) scale(0.95) !important;
		opacity: 0 !important;
		clip-path: inset(0 0 0 100%) !important;
		transition: opacity 0.5s cubic-bezier(0.25, 1, 0.5, 1), 
		            transform 0.5s cubic-bezier(0.25, 1, 0.5, 1),
		            clip-path 0.6s cubic-bezier(0.25, 1, 0.5, 1),
		            background-color 0.3s ease,
		            border-color 0.3s ease,
		            box-shadow 0.3s ease !important;
	}

	:global(.overview-hotspot.hotspot-left.visible:hover .overview-hotspot-label) {
		opacity: 1 !important;
		transform: translateX(0) scale(1) !important;
		clip-path: inset(0 0 0 0) !important;
	}

	/* Premium Hover effect */
	:global(.overview-hotspot.hotspot-right .overview-hotspot-label:hover) {
		border-color: rgba(176, 141, 79, 0.7) !important;
		box-shadow: 0 12px 36px rgba(176, 141, 79, 0.25), 0 6px 16px rgba(0, 0, 0, 0.12) !important;
		transform: translateX(2px) scale(1.03) !important;
	}

	:global(.overview-hotspot.hotspot-left .overview-hotspot-label:hover) {
		border-color: rgba(176, 141, 79, 0.7) !important;
		box-shadow: 0 12px 36px rgba(176, 141, 79, 0.25), 0 6px 16px rgba(0, 0, 0, 0.12) !important;
		transform: translateX(-2px) scale(1.03) !important;
	}

	@media (max-width: 768px) {
		:global(.overview-hotspot .overview-hotspot-label) {
			padding: 4px 16px 4px 4px !important;
			gap: 10px !important;
		}
		:global(.overview-hotspot .hotspot-label-img) {
			width: 44px !important;
			height: 44px !important;
		}
		:global(.overview-hotspot .hotspot-label-title) {
			font-size: 13px !important;
		}
		:global(.overview-hotspot .hotspot-label-subtitle) {
			font-size: 10px !important;
		}
	}

</style>
