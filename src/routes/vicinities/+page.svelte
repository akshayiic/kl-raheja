<script>
	import { goto } from '$app/navigation';
	import { getContext, onMount, onDestroy } from 'svelte';
	import minimizeBtn from '$lib/images/minimize-icon.svg';
	import maximizeBtn from '$lib/images/maximize-icon.svg';
	import * as Accordion from '$lib/components/ui/accordion/index.ts';
	import { createLoadObserver } from '$lib/utils.ts';
	import { writable } from 'svelte/store';
	import { slide } from 'svelte/transition';
	import { browser } from '$app/environment';

	// Video asset imports for Cafes & Clubs
	import bastianVid from '$lib/videos/Bastian.mp4';
	import kharVid from '$lib/videos/Khar Gymkhana.mp4';
	import migVid from '$lib/videos/MIG.mp4';
	import nsciVid from '$lib/videos/National Sports Club of India.mp4';
	import ottersVid from '$lib/videos/Otters.mp4';
	import paliVid from '$lib/videos/Pali Village Cafe.mp4';
	import tajVid from '$lib/videos/Taj lounges.mp4';
	import totosVid from '$lib/videos/Totos.mp4';
	import veronicasVid from '$lib/videos/Veronicas Pali Hill Cafe.mp4';
	import willingdonVid from '$lib/videos/Willingdon Club.mp4';
	import introVid from '$lib/videos/introvideo.mp4';

	let currentUI = getContext('currentUI');
	let cleanId;
	$currentUI['vicinity'] = true;
	const onload = createLoadObserver(() => {
		console.log('loaded!!!');
	});

	const isSafariMobile =
		browser &&
		/iPhone|iPad|iPod/.test(navigator.userAgent) &&
		/Safari/.test(navigator.userAgent) &&
		!/CriOS|FxiOS|OPiOS|EdgiOS/.test(navigator.userAgent);

	const isAmenitiesMinimized = writable();
	$: isAmenitiesMinimized.set(isSafariMobile ? true : false);

	let vicinityImg = getContext('vicinityImg');
	let isLoaded = true;
	let activeSrc = '';
	let prevSrc = '';
	let imgEl;
	let videoLoading = true;

	$: {
		if (activeVideo) {
			videoLoading = true;
		}
	}

	$: displaySrc = $vicinityImg != '-' ? 'https://assets.vestate.io/webtool/kraheja/kraheja/vicinities/' + $vicinityImg : '';
	$: {
		if (displaySrc && displaySrc !== activeSrc) {
			prevSrc = activeSrc;
			activeSrc = displaySrc;
			isLoaded = false;
		}
	}
	$: if (imgEl && imgEl.complete && imgEl.naturalWidth > 0) {
		isLoaded = true;
	}



	onMount(async () => {
		vicinityImg.set('-');
		console.log('vicinity mounted');
	});

	let activeFolder = '';
	let activeCategoryFolder = '';
	let activeLabel = '';
	let activeDistance = '';
	let activeVideo = '';
	let cardSlideIndex = 1;
	let galleryInterval;

	function startGalleryTimer() {
		stopGalleryTimer();
		galleryInterval = setInterval(() => {
			cardSlideIndex = cardSlideIndex === 3 ? 1 : cardSlideIndex + 1;
		}, 3000);
	}

	function stopGalleryTimer() {
		if (galleryInterval) {
			clearInterval(galleryInterval);
			galleryInterval = null;
		}
	}

	function selectSlide(num) {
		cardSlideIndex = num;
		startGalleryTimer();
	}

	$: {
		if (activeFolder && activeCategoryFolder) {
			startGalleryTimer();
		} else {
			stopGalleryTimer();
		}
	}

	$: if ($vicinityImg) {
		cardSlideIndex = 1;
	}

	onDestroy(() => {
		stopGalleryTimer();
	});

	$: {
		activeFolder = '';
		activeCategoryFolder = '';
		activeLabel = '';
		activeDistance = '';
		activeVideo = '';

		for (const cat of vicinityCategories) {
			const found = cat.items.find((item) => item.id === $vicinityImg);
			if (found) {
				activeFolder = found.folder || '';
				activeCategoryFolder = cat.folder || '';
				activeLabel = found.label || '';
				activeDistance = found.distance || '';
				if (found.video) {
					activeVideo = found.video;
				} else if (found.folder && cat.folder) {
					// Encode folder names for URL safety
					const encodedCat = cat.folder;
					const encodedSub = encodeURIComponent(found.videoName || found.folder);
					activeVideo = `https://assets.vestate.io/kl-rahega/videos/${encodedCat}/${encodedSub}.mp4`;
				}
				break;
			}
		}
	}

	let lightboxOpen = false;
	let lightboxIndex = 1;

	function openLightbox(index) {
		lightboxIndex = index;
		lightboxOpen = true;
	}

	function closeLightbox() {
		lightboxOpen = false;
	}

	function nextImage() {
		lightboxIndex = lightboxIndex === 3 ? 1 : lightboxIndex + 1;
	}

	function prevImage() {
		lightboxIndex = lightboxIndex === 1 ? 3 : lightboxIndex - 1;
	}

	function handleKeyDown(e) {
		if (!lightboxOpen) return;
		if (e.key === 'ArrowRight') nextImage();
		if (e.key === 'ArrowLeft') prevImage();
		if (e.key === 'Escape') closeLightbox();
	}

	// Organized vicinities from local folder structure
	const isDay = writable();

	// Vicinity data organized by category
	const vicinityCategories = [
		{
			id: 'connectivity',
			name: 'Connectivity',
			folder: 'Connectivity',
			items: [
				{ id: 'Connectivity/BandraTerminus.webp', label: 'Bandra Terminus', folder: 'Bandra Terminus', distance: '2.5 KM', subcategory: 'Railway' },
				{ id: 'Connectivity/BKCMetrostation.webp', label: 'BKC Metro Station', folder: 'BKC Metro station', videoName: 'BKC Metro Station', distance: '1.8 KM', subcategory: 'Metro' },
				{ id: 'Connectivity/SantacruzMetroLine.webp', label: 'Santa Cruz Metro Line', folder: 'Santacruz Metro line 3', videoName: 'Santacruz Metro Line 3', distance: '2.1 KM', subcategory: 'Metro' },
				{ id: 'Connectivity/CoastalRoad.webp', label: 'Coastal Road', folder: 'Coastal Road', distance: '4.2 KM', subcategory: 'Roadways' },
				{ id: 'Connectivity/Versova Bandra Sea Link.webp', label: 'Versova Bandra Sea Link', folder: 'Versova-Bandra sea link', videoName: 'Versova', distance: '3.5 KM', subcategory: 'Roadways' },
				{ id: 'Connectivity/WesternExpressHighway.webp', label: 'Western Express Highway', folder: 'Western Express Highway', distance: '1.0 KM', subcategory: 'Roadways' },
				{ id: 'Connectivity/WorliSeaLink.webp', label: 'Worli Sea Link', folder: 'Bandra Worli Sea Link', distance: '3.8 KM', subcategory: 'Roadways' }
			]
		},
		{
			id: 'cafe-club',
			name: 'Cafes & Clubs',
			folder: 'Cafe%20%26%20Clubs',
			items: [
				{ id: 'Cafe%20and%20Club/Bastian.webp', label: 'Bastian', video: bastianVid, folder: 'Bastian', distance: '1.2 KM' },
				{ id: 'Cafe%20and%20Club/Khar Gymkhana.webp', label: 'Khar Gymkhana', video: kharVid, folder: 'Khar Gymkhana', distance: '0.8 KM' },
				{ id: 'Cafe%20and%20Club/MIG.webp', label: 'MIG', video: migVid, folder: 'MIG', distance: '1.5 KM' },
				{ id: 'Cafe%20and%20Club/NationalSportsClubofIndia.webp', label: 'National Sports Club of India', video: nsciVid, folder: 'National Sports Club of India', distance: '5.0 KM' },
				{ id: 'Cafe%20and%20Club/Otters.webp', label: 'Otters', video: ottersVid, folder: 'Otter_s', distance: '1.1 KM' },
				{ id: 'Cafe%20and%20Club/PaliVillageCafé.webp', label: 'Pali Village Café', video: paliVid, folder: 'Pali Village Cafe', distance: '0.9 KM' },
				{ id: 'Cafe%20and%20Club/Taj lounges .webp', label: 'Taj Lounges', video: tajVid, folder: 'Taj Lounges', distance: '4.5 KM' },
				{ id: 'Cafe%20and%20Club/Toto\'s.webp', label: 'Toto\'s', video: totosVid, folder: 'Toto_s', distance: '0.7 KM' },
				{ id: 'Cafe%20and%20Club/Veronicas Pali Hill Cafes.webp', label: 'Veronicas Pali Hill Cafes', video: veronicasVid, folder: 'Veronica_s-Pali Hill Cafes', distance: '0.6 KM' },
				{ id: 'Cafe%20and%20Club/WillingdonClub .webp', label: 'Willingdon Club', video: willingdonVid, folder: 'Willingdon club', distance: '6.2 KM' }
			]
		},
		{
			id: 'commercial',
			name: 'Commercial',
			folder: 'Commercial',
			items: [
				{ id: 'Commercial/BKC  .webp', label: 'BKC', folder: 'BKC', distance: '1.8 KM' },
				{ id: 'Commercial/JioWorldCentre.webp', label: 'Jio World Centre', folder: 'Jio World Centre', distance: '2.0 KM' },
				{ id: 'Commercial/LowerParel.webp', label: 'Lower Parel', folder: 'Lower Parel', distance: '8.5 KM' },
				{ id: 'Commercial/NMAC .webp', label: 'NMAC', folder: 'NMAC', videoName: 'NMACC (1)', distance: '2.0 KM' },
				{ id: 'Commercial/Worli.webp', label: 'Worli', folder: 'Worli', distance: '9.0 KM' }
			]
		},
		{
			id: 'hospital',
			name: 'Hospitals',
			folder: 'Hospitals',
			items: [
				{ id: 'Hospital/Asian Heart Institute.webp', label: 'Asian Heart Institute', folder: 'Asian Heart Institute', distance: '2.2 KM' },
				{ id: 'Hospital/HolyFamily  .webp', label: 'Holy Family', folder: 'Holy Family Hospital', distance: '1.6 KM' },
				{ id: 'Hospital/LilavatiHospital.webp', label: 'Lilavati Hospital', folder: 'Lilavati Hospital', videoName: 'Lilavati Hospital & Research Centre', distance: '1.3 KM' }
			]
		},
		{
			id: 'education',
			name: 'Education Institutes',
			folder: 'Education%20Institutes',
			items: [
				{ id: 'Education/AmericanSchoolofBombay.webp', label: 'American School of Bombay', folder: 'American School of Bombay', distance: '2.4 KM' },
				{ id: 'Education/DhirubhaiAmbani.webp', label: 'Dhirubhai Ambani', folder: 'Dhirubhai Ambani International School', distance: '2.2 KM' },
				{ id: 'Education/StAndrews .webp', label: 'St. Andrews', folder: 'St. Andrews College', videoName: 'St. Andrew_s College', distance: '1.1 KM' },
				{ id: 'Education/StStanislaus.webp', label: 'St. Stanislaus', folder: 'St. Stanislaus International School', videoName: 'St. Stanislaus High School', distance: '0.8 KM' }
			]
		},
		{
			id: 'faith-heritage',
			name: 'Faith & Heritage',
			folder: 'Faith%20%26%20Heritage',
			items: [
				{ id: 'Faith%20And%20Heritage/Mount Mary Basilica.webp', label: 'Mount Mary Basilica', folder: 'Mount Mary', videoName: 'Mount Mary Basilica', distance: '1.7 KM' }
			]
		},
		{
			id: 'retail-lifestyle',
			name: 'Retail & Lifestyle',
			folder: 'Retail%20%26%20Lifestyle',
			items: [
				{ id: 'Retail%20And%20Lifestyle/Bandstand.webp', label: 'Bandstand', folder: 'Bandstand', distance: '1.4 KM' },
				{ id: 'Retail%20And%20Lifestyle/CarterRoad.webp', label: 'Carter Road', folder: 'Carter Road', distance: '1.5 KM' },
				{ id: 'Retail%20And%20Lifestyle/JioWorldDrive.webp', label: 'Jio World Drive', folder: 'Jio World Drive', distance: '2.3 KM' },
				{ id: 'Retail%20And%20Lifestyle/LinkingHillRoad .webp', label: 'Linking Hill Road', folder: 'Linking-Hill Road', videoName: 'Linking Hill Road', distance: '0.5 KM' }
			]
		}
	];

	function toggleDayNight() {
		isDay.update((value) => !value);
		isDay.subscribe((value) => {
			// hotspotName.set(value ? 'night' : 'Exterior');
		});
	}
</script>

{#if $isAmenitiesMinimized}
	<button
		class="minimized-sidebar-tab"
		style="position: fixed; left: 0; top: 50%; transform: translateY(-50%); z-index: 1001; margin-top: 0;"
		on:click={() => ($isAmenitiesMinimized = false)}
		type="button"
		id="minimize-toggle-vicinity"
	>
		<span class="chevron-group">
			<svg width="24" height="24" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
				<path d="M7 6L13 12L7 18" stroke="rgba(255, 255, 255, 0.35)" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round"/>
				<path d="M14 6L20 12L14 18" stroke="rgba(255, 255, 255, 0.85)" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round"/>
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
{:else}
	<div class="left-panel-wrapper">
		<div class="left-panel p-2">
			<div class="left-panel--header flex items-center justify-between">
				<div class="header-icon-box">
					<svg width="22" height="22" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
						<polygon points="3 6 9 3 15 6 21 3 21 18 15 21 9 18 3 21"></polygon>
						<line x1="9" y1="3" x2="9" y2="18"></line>
						<line x1="15" y1="6" x2="15" y2="21"></line>
					</svg>
				</div>
				<div class="left-title flex flex-col justify-center ml-3 text-left">
					<span class="explore-subtitle">Click to Explore</span>
					<span class="explore-title">Locations</span>
				</div>
				<button
					on:click={() => {
						$isAmenitiesMinimized = true;
					}}
					class="ghost-btn close-btn ml-auto !px-0 !py-0"
					id="minimize-toggle-vicinity"
				>
					<svg width="24" height="24" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
						<path d="M17 18L11 12L17 6" stroke="rgba(255, 255, 255, 0.85)" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round"/>
						<path d="M10 18L4 12L10 6" stroke="rgba(255, 255, 255, 0.35)" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round"/>
					</svg>
				</button>
			</div>

			<div class="panel-divider"></div>

			<div class="infrastructure-heading">Current Infrastructure</div>

			<div class="no-hovers">
				<div class="inner-btn-group">
					<Accordion.Root class="w-full sm:max-w-full" multiple={true}>
						<Accordion.Item class="hidden" value="item-1wqweqweqweqwe">
							<Accordion.Trigger id="station-level-ss" class="hidden">asdasdasd</Accordion.Trigger>
						</Accordion.Item>

						{#each vicinityCategories as category}
							<Accordion.Item value={category.id}>
								<Accordion.Trigger id={category.id + '-level'}>
									<div class="flex items-center gap-3 w-full text-left">
										{#if category.id === 'connectivity'}
											<svg class="category-icon" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
												<path d="M3 12h18" />
												<path d="M3 18h18" />
												<path d="M6 12v6" />
												<path d="M18 12v6" />
												<path d="M12 12v6" />
												<path d="M3 12c3-4 6-4 9-4s6 0 9 4" />
											</svg>
										{:else if category.id === 'cafe-club'}
											<svg class="category-icon" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
												<path d="M12 22C17.5228 22 22 17.5228 22 12C22 9.5 20 8.5 20 6.5C20 4.5 18 3 15 3C10.5 3 4 6.5 4 12C4 17.5228 7.47715 22 12 22Z" />
												<circle cx="7.5" cy="10.5" r="1.5" fill="currentColor" />
												<circle cx="11.5" cy="7.5" r="1.5" fill="currentColor" />
												<circle cx="16.5" cy="9.5" r="1.5" fill="currentColor" />
											</svg>
										{:else if category.id === 'commercial'}
											<svg class="category-icon" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
												<rect x="4" y="2" width="16" height="20" rx="2" ry="2" />
												<line x1="9" y1="22" x2="9" y2="16" />
												<line x1="15" y1="22" x2="15" y2="16" />
												<line x1="9" y1="16" x2="15" y2="16" />
												<path d="M8 6h2v2H8V6zm6 0h2v2h-2V6zm-6 5h2v2H8v-2zm6 0h2v2h-2v-2z" />
											</svg>
										{:else if category.id === 'hospital'}
											<svg class="category-icon" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
												<path d="M3 9l9-7 9 7v11a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2z" />
												<line x1="12" y1="9" x2="12" y2="15" />
												<line x1="9" y1="12" x2="15" y2="12" />
											</svg>
										{:else if category.id === 'education'}
											<svg class="category-icon" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
												<path d="M22 10v6M2 10l10-5 10 5-10 5z" />
												<path d="M6 12v5c0 2 2 3 6 3s6-1 6-3v-5" />
											</svg>
										{:else if category.id === 'faith-heritage'}
											<svg class="category-icon" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
												<path d="M4 22h16" />
												<path d="M20 18H4v-2h16v2z" />
												<path d="M12 2L2 7v3h20V7L12 2z" />
												<line x1="6" y1="10" x2="6" y2="16" />
												<line x1="10" y1="10" x2="10" y2="16" />
												<line x1="14" y1="10" x2="14" y2="16" />
												<line x1="18" y1="10" x2="18" y2="16" />
											</svg>
										{:else if category.id === 'retail-lifestyle'}
											<svg class="category-icon" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
												<path d="M6 2L3 6v14a2 2 0 0 0 2 2h14a2 2 0 0 0 2-2V6l-3-4z" />
												<line x1="3" y1="6" x2="21" y2="6" />
												<path d="M16 10a4 4 0 0 1-8 0" />
											</svg>
										{/if}
										<span class="category-name">{category.name === 'Cafes & Clubs' ? 'Cafe & Clubs' : category.name}</span>
									</div>
								</Accordion.Trigger>
								<Accordion.Content>
									{#each category.items as item}
										<button
											class={$vicinityImg == item.id
												? 'active inner-modal-btn'
												: 'inner-modal-btn'}
											id={'x-' + item.id.replaceAll(/[\s./]+/g, '-') + '-am'}
											on:click={() => vicinityImg.set(item.id)}
										>
											{item.label}
										</button>
									{/each}
								</Accordion.Content>
							</Accordion.Item>
						{/each}
					</Accordion.Root>
				</div>
			</div>
		</div>
	</div>
{/if}

<div class="d-block visible absolute bottom-0 left-0 right-0 top-0">
	{#if $vicinityImg === '-'}
		<video
			src={introVid}
			autoplay
			muted
			playsinline
			class="absolute top-0 left-0 h-full w-full object-cover z-40"
		></video>
	{/if}

	{#if $vicinityImg != '-'}
		<div class="absolute top-0 left-0 w-full h-full z-50 bg-black overflow-hidden">
			{#if activeVideo}
				{#if videoLoading}
					<div class="absolute inset-0 z-50 flex flex-col items-center justify-center bg-black/60 backdrop-blur-md transition-all duration-300">
						<!-- Premium custom luxury gold spinner -->
						<div class="relative w-16 h-16 mb-4 animate-fade-in">
							<!-- Spinner track -->
							<div class="absolute inset-0 rounded-full border-4 border-white/10"></div>
							<!-- Spinner active line -->
							<div class="absolute inset-0 rounded-full border-4 border-t-[#DEAD66] border-r-transparent border-b-transparent border-l-transparent animate-spin"></div>
						</div>
						<span class="text-white/85 text-[10px] uppercase tracking-[0.2em] font-light font-mono animate-pulse">
							LOADING VIDEO...
						</span>
					</div>
				{/if}

				<video
					src={activeVideo}
					autoplay
					muted
					playsinline
					class="absolute top-0 left-0 h-full w-full object-cover transition-opacity duration-300 {videoLoading ? 'opacity-0' : 'opacity-100'}"
					on:playing={() => videoLoading = false}
					on:canplay={() => videoLoading = false}
					on:loadstart={() => videoLoading = true}
					on:waiting={() => videoLoading = true}
				></video>
			{:else}
				{#if !isLoaded && prevSrc}
					<img
						src={prevSrc}
						alt="Vicinity Previous"
						class="absolute top-0 left-0 h-full w-full object-cover filter blur-[8px] scale-105 pointer-events-none"
					/>
				{/if}

				<img
					bind:this={imgEl}
					src={activeSrc}
					alt="Vicinity"
					class="absolute top-0 left-0 h-full w-full object-cover transition-opacity duration-300 {isLoaded ? 'opacity-100' : 'opacity-0'}"
					on:load={() => { isLoaded = true; }}
					on:error={() => { isLoaded = true; }}
				/>
			{/if}
		</div>
	{/if}
</div>

<!-- Right-side gallery card (preview with dots) -->
{#if activeFolder && activeCategoryFolder}
	{@const activePreviewUrl = `https://assets.vestate.io/kl-rahega/images/${activeCategoryFolder}/${encodeURIComponent(activeFolder)}/${cardSlideIndex}.png`}
	<div class="fixed right-6 top-1/2 -translate-y-1/2 z-[1000] w-[345px] bg-[#0c1c28]/70 backdrop-blur-2xl border border-white/10 rounded-[28px] shadow-2xl p-5 select-none animate-fade-in text-left">
		<!-- Header with Title & Distance & Close Button -->
		<div class="flex justify-between items-start mb-4">
			<div class="flex flex-col pr-4">
				<h3 class="text-white text-[19px] font-semibold tracking-wide leading-tight uppercase font-serif" style="font-family: 'The Seasons', serif;">
					{activeLabel}
				</h3>
				{#if activeDistance}
					<div class="flex items-center gap-1.5 text-white/50 text-[11px] mt-1.5 font-medium">
						<!-- Custom direction/road split icon -->
						<svg class="w-4 h-4 text-white/60" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
							<rect x="3" y="3" width="18" height="18" rx="2" ry="2"></rect>
							<line x1="9" y1="3" x2="9" y2="21"></line>
							<line x1="15" y1="3" x2="15" y2="21"></line>
							<line x1="3" y1="9" x2="21" y2="9"></line>
							<line x1="3" y1="15" x2="21" y2="15"></line>
						</svg>
						{activeDistance}
					</div>
				{/if}
			</div>

			<!-- Close button -->
			<button
				class="text-white/60 hover:text-white cursor-pointer transition-colors p-1.5 rounded-full hover:bg-white/10 border border-white/10 flex items-center justify-center w-8 h-8"
				on:click={() => vicinityImg.set('-')}
				type="button"
			>
				<svg class="w-4 h-4" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round">
					<line x1="18" y1="6" x2="6" y2="18"></line>
					<line x1="6" y1="6" x2="18" y2="18"></line>
				</svg>
			</button>
		</div>

		<!-- Image Container with dots overlay -->
		<div class="relative w-full h-[190px] rounded-[18px] overflow-hidden border border-white/10 group cursor-pointer shadow-lg">
			<!-- Image click opens lightbox -->
			<div class="w-full h-full" on:click={() => openLightbox(cardSlideIndex)}>
				<img
					src={activePreviewUrl}
					alt={activeLabel}
					class="w-full h-full object-cover group-hover:scale-105 transition-transform duration-700"
				/>
				<div class="absolute inset-0 bg-black/5 group-hover:bg-black/0 transition-colors"></div>
			</div>

			<!-- Indicator dots centered at the bottom of the image -->
			<div class="absolute bottom-4 left-0 right-0 flex justify-center gap-2.5 z-10" on:click|stopPropagation>
				{#each [1, 2, 3] as imgNum}
					<button
						class="w-2 h-2 rounded-full transition-all duration-300 p-0 cursor-pointer {cardSlideIndex === imgNum ? 'bg-white scale-125' : 'bg-white/40 hover:bg-white/70'}"
						on:click={() => selectSlide(imgNum)}
						type="button"
						aria-label={`Slide ${imgNum}`}
					></button>
				{/each}
			</div>
		</div>
	</div>
{/if}

<!-- Center Lightbox Overlay -->
{#if lightboxOpen && activeFolder && activeCategoryFolder}
	<div class="lightbox-overlay-glass fixed inset-0 z-[10000] flex flex-col items-center justify-center animate-fade-in" on:click={closeLightbox}>
		
		<!-- Viewport close button (top right) -->
		<button
			class="lightbox-close-glass absolute top-6 right-6 cursor-pointer transition-all flex items-center justify-center w-11 h-11 rounded-full shadow-lg"
			on:click={closeLightbox}
			type="button"
		>
			<svg class="w-5 h-5" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round">
				<line x1="18" y1="6" x2="6" y2="18"></line>
				<line x1="6" y1="6" x2="18" y2="18"></line>
			</svg>
		</button>

		<!-- Main Image Container -->
		<div 
			class="lightbox-card-glass relative max-w-5xl w-[85vw] h-[65vh] rounded-[28px] overflow-hidden shadow-2xl flex items-center justify-center p-3"
			on:click|stopPropagation
		>
			<img
				src={`https://assets.vestate.io/kl-rahega/images/${activeCategoryFolder}/${encodeURIComponent(activeFolder)}/${lightboxIndex}.png`}
				alt={`${activeLabel} Gallery`}
				class="w-full h-full object-cover rounded-[20px] select-none"
			/>
		</div>

		<!-- Navigation Pill below the Image -->
		<div 
			class="lightbox-nav-glass flex items-center gap-6 mt-6 select-none px-6 py-2.5 rounded-full shadow-lg"
			on:click|stopPropagation
		>
			<!-- Prev Button -->
			<button
				class="text-white/50 hover:text-white cursor-pointer transition-all p-1 rounded-full hover:bg-white/10 flex items-center justify-center w-8 h-8"
				on:click={prevImage}
				type="button"
			>
				<svg class="w-5 h-5" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="3" stroke-linecap="round" stroke-linejoin="round">
					<polyline points="15 18 9 12 15 6"></polyline>
				</svg>
			</button>

			<!-- Page count text -->
			<span class="text-white/80 text-xs font-semibold tracking-widest font-mono">
				{lightboxIndex} / 3
			</span>

			<!-- Next Button -->
			<button
				class="text-white/50 hover:text-white cursor-pointer transition-all p-1 rounded-full hover:bg-white/10 flex items-center justify-center w-8 h-8"
				on:click={nextImage}
				type="button"
			>
				<svg class="w-5 h-5" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="3" stroke-linecap="round" stroke-linejoin="round">
					<polyline points="9 18 15 12 9 6"></polyline>
				</svg>
			</button>
		</div>
	</div>
{/if}

<svelte:window on:keydown={handleKeyDown} />

<!-- Go Back Button at bottom-left corner -->
<button
	class="go-back-btn"
	on:click={() => {
		$currentUI.vicinity = false;
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

<style>
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

	:global(.left-panel-wrapper) {
		bottom: 6.5rem !important;
	}
	@media (min-width: 1020px) {
		:global(.left-panel-wrapper) {
			bottom: 8.5rem !important;
		}
	}

	/* Lightbox Glassmorphism Overlay styling */
	.lightbox-overlay-glass {
		background: rgba(10, 22, 34, 0.5) !important;
		backdrop-filter: blur(35px) saturate(210%) !important;
		-webkit-backdrop-filter: blur(35px) saturate(210%) !important;
	}

	/* Glass Close Button */
	.lightbox-close-glass {
		background: rgba(255, 255, 255, 0.06) !important;
		backdrop-filter: blur(10px) !important;
		-webkit-backdrop-filter: blur(10px) !important;
		border: 1px solid rgba(255, 255, 255, 0.15) !important;
		color: rgba(255, 255, 255, 0.6) !important;
		box-shadow: 0 8px 32px 0 rgba(0, 0, 0, 0.25) !important;
	}

	.lightbox-close-glass:hover {
		color: #ffffff !important;
		background: rgba(255, 255, 255, 0.12) !important;
		border-color: rgba(255, 255, 255, 0.25) !important;
		transform: scale(1.05);
	}

	/* Glass Card containing the image */
	.lightbox-card-glass {
		background: rgba(255, 255, 255, 0.05) !important;
		backdrop-filter: blur(25px) saturate(180%) !important;
		-webkit-backdrop-filter: blur(25px) saturate(180%) !important;
		border: 1px solid rgba(255, 255, 255, 0.12) !important;
		box-shadow: 0 25px 50px -12px rgba(0, 0, 0, 0.5), 0 0 40px rgba(255, 255, 255, 0.05) !important;
	}

	/* Glass Navigation Pill */
	.lightbox-nav-glass {
		background: rgba(255, 255, 255, 0.07) !important;
		backdrop-filter: blur(20px) !important;
		-webkit-backdrop-filter: blur(20px) !important;
		border: 1px solid rgba(255, 255, 255, 0.15) !important;
		box-shadow: 0 10px 30px rgba(0, 0, 0, 0.3) !important;
	}
</style>
