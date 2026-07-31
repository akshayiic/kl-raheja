<script>
	import { goto } from '$app/navigation';
	import { getContext, onMount } from 'svelte';
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

	$: if ($vicinityImg) {
		cardSlideIndex = 1;
	}

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

<div class="left-panel-wrapper">
	<div class="left-panel p-2">
		<div class="left-panel--header flex justify-between gap-[2rem] lg:gap-[5rem]">
			<div class="left-title flex items-center font-bold">
				<svg
					class="mr-2"
					width="17"
					height="16"
					viewBox="0 0 17 16"
					fill="none"
					xmlns="http://www.w3.org/2000/svg"
				>
					<path
						d="M8.49535 0.0693359C11.2918 0.0693359 13.4186 2.10511 13.4992 4.65137C13.524 5.34636 13.3008 5.986 13.0341 6.60718C12.3087 8.32929 11.2918 9.87918 10.2315 11.4106C9.79127 12.0441 9.32622 12.653 8.87359 13.2803C8.78058 13.4095 8.71857 13.4464 8.60696 13.2926C7.05063 11.2077 5.5687 9.07964 4.46501 6.71174C3.37992 4.38075 4.31 2.43723 5.4943 1.34246C6.40578 0.499862 7.49087 0.0877871 8.49535 0.0693359ZM8.74338 6.36117C9.61145 6.36117 10.3183 5.66618 10.3245 4.80512C10.3307 3.94407 9.60525 3.21833 8.73717 3.21833C7.8691 3.21833 7.14364 3.95022 7.15604 4.81128C7.16844 5.67233 7.8753 6.36117 8.74338 6.36117Z"
						fill="#5A4DE3"
					/>
					<path
						d="M8.29317 14.232C10.3207 14.2258 11.8585 14.0167 13.3218 13.414C13.8488 13.1987 14.3511 12.9342 14.7541 12.5222C15.2006 12.067 15.1944 11.6611 14.7417 11.2121C14.3015 10.7754 13.7434 10.5233 13.173 10.308C12.9808 10.2342 12.9684 10.1789 13.0738 10.0128C13.2784 9.68068 13.4582 9.33626 13.6442 8.99184C13.7124 8.86883 13.762 8.80118 13.9232 8.86883C14.6735 9.1948 15.3866 9.58842 15.9508 10.1912C16.9305 11.2429 16.9243 12.6021 15.9198 13.6354C15.1199 14.4534 14.1031 14.9085 13.018 15.2037C9.99211 16.034 6.96625 16.0832 3.9652 15.0684C3.15913 14.7978 2.41506 14.398 1.77641 13.826C0.4805 12.6513 0.474299 11.1445 1.75781 9.9513C2.28485 9.46542 2.89871 9.11485 3.54356 8.80733C3.71098 8.72737 3.76678 8.78272 3.84119 8.92418C4.0272 9.2809 4.21322 9.63148 4.42404 9.96975C4.51704 10.1235 4.47984 10.1666 4.33103 10.2281C3.89079 10.4126 3.46915 10.6278 3.08472 10.9046C2.97931 10.9846 2.8739 11.0707 2.7747 11.1629C2.25385 11.6611 2.24765 12.0916 2.7809 12.5775C3.52496 13.2602 4.44884 13.58 5.40372 13.8322C6.50741 14.1274 7.62351 14.2443 8.29317 14.232Z"
						fill="#5A4DE3"
					/>
				</svg>
				Vicinities
			</div>
			<button
				on:click={() => {
					$isAmenitiesMinimized = !$isAmenitiesMinimized;
				}}
				class="ghost-btn close-btn !px-0 !py-0"
				id="minimize-toggle-vicinity"
			>
				{#if !$isAmenitiesMinimized}
					<img src={minimizeBtn} alt="minimize" />
				{/if}
				{#if $isAmenitiesMinimized}
					<img src={maximizeBtn} alt="maximize" />
				{/if}
			</button>
		</div>
		<div
			class={!$isAmenitiesMinimized ? 'block' : 'hidden'}
			isInteriorUnitDataMinimized
			transition:slide={{ duration: 100, axis: 'y' }}
		>
			<div class="no-hovers pt-3">
				<div class="inner-btn-group">
					<Accordion.Root class="w-full sm:max-w-full" multiple={true}>
						<Accordion.Item class="hidden" value="item-1wqweqweqweqwe">
							<Accordion.Trigger id="station-level-ss" class="hidden">asdasdasd</Accordion.Trigger>
						</Accordion.Item>

						{#each vicinityCategories as category}
							<Accordion.Item value={category.id}>
								<Accordion.Trigger id={category.id + '-level'}>{category.name}</Accordion.Trigger>
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
</div>

<div class="d-block visible absolute bottom-0 left-0 right-0 top-0">
	{#if $vicinityImg === '-'}
		<video
			src={introVid}
			autoplay
			loop
			muted
			playsinline
			class="absolute top-0 left-0 h-full w-full object-cover z-40"
		></video>
	{/if}

	{#if $vicinityImg != '-'}
		<div class="absolute top-0 left-0 w-full h-full z-50 bg-black overflow-hidden">
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

			{#if activeVideo}
				<video
					src={activeVideo}
					autoplay
					loop
					muted
					playsinline
					class="absolute top-0 left-0 h-full w-full object-cover"
				></video>
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
						on:click={() => cardSlideIndex = imgNum}
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
	<div class="fixed inset-0 bg-black/90 backdrop-blur-xl z-[10000] flex flex-col items-center justify-center animate-fade-in" on:click={closeLightbox}>
		
		<!-- Viewport close button (top right) -->
		<button
			class="absolute top-6 right-6 text-white/50 hover:text-white cursor-pointer transition-colors p-2.5 rounded-full hover:bg-white/10 border border-white/10 flex items-center justify-center w-10 h-10 shadow-lg"
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
			class="relative max-w-5xl w-[85vw] h-[65vh] rounded-[24px] overflow-hidden border border-white/10 bg-black/20 shadow-2xl flex items-center justify-center"
			on:click|stopPropagation
		>
			<img
				src={`https://assets.vestate.io/kl-rahega/images/${activeCategoryFolder}/${encodeURIComponent(activeFolder)}/${lightboxIndex}.png`}
				alt={`${activeLabel} Gallery`}
				class="w-full h-full object-cover select-none"
			/>
		</div>

		<!-- Navigation Pill below the Image -->
		<div 
			class="flex items-center gap-6 mt-6 select-none bg-black/40 backdrop-blur-md px-5 py-2 rounded-full border border-white/10 shadow-lg"
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
</style>
