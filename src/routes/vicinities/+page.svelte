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
	let activeVideo = '';

	$: {
		activeFolder = '';
		activeCategoryFolder = '';
		activeLabel = '';
		activeVideo = '';

		for (const cat of vicinityCategories) {
			const found = cat.items.find((item) => item.id === $vicinityImg);
			if (found) {
				activeFolder = found.folder || '';
				activeCategoryFolder = cat.folder || '';
				activeLabel = found.label || '';
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
				{ id: 'Connectivity/BandraTerminus.webp', label: 'Bandra Terminus', folder: 'Bandra Terminus', subcategory: 'Railway' },
				{ id: 'Connectivity/BKCMetrostation.webp', label: 'BKC Metro Station', folder: 'BKC Metro station', videoName: 'BKC Metro Station', subcategory: 'Metro' },
				{ id: 'Connectivity/SantacruzMetroLine.webp', label: 'Santa Cruz Metro Line', folder: 'Santacruz Metro line 3', videoName: 'Santacruz Metro Line 3', subcategory: 'Metro' },
				{ id: 'Connectivity/CoastalRoad.webp', label: 'Coastal Road', folder: 'Coastal Road', subcategory: 'Roadways' },
				{ id: 'Connectivity/Versova Bandra Sea Link.webp', label: 'Versova Bandra Sea Link', folder: 'Versova-Bandra sea link', videoName: 'Versova', subcategory: 'Roadways' },
				{ id: 'Connectivity/WesternExpressHighway.webp', label: 'Western Express Highway', folder: 'Western Express Highway', subcategory: 'Roadways' },
				{ id: 'Connectivity/WorliSeaLink.webp', label: 'Worli Sea Link', folder: 'Bandra Worli Sea Link', subcategory: 'Roadways' }
			]
		},
		{
			id: 'cafe-club',
			name: 'Cafes & Clubs',
			folder: 'Cafe%20%26%20Clubs',
			items: [
				{ id: 'Cafe%20and%20Club/Bastian.webp', label: 'Bastian', video: bastianVid, folder: 'Bastian' },
				{ id: 'Cafe%20and%20Club/Khar Gymkhana.webp', label: 'Khar Gymkhana', video: kharVid, folder: 'Khar Gymkhana' },
				{ id: 'Cafe%20and%20Club/MIG.webp', label: 'MIG', video: migVid, folder: 'MIG' },
				{ id: 'Cafe%20and%20Club/NationalSportsClubofIndia.webp', label: 'National Sports Club of India', video: nsciVid, folder: 'National Sports Club of India' },
				{ id: 'Cafe%20and%20Club/Otters.webp', label: 'Otters', video: ottersVid, folder: 'Otter_s' },
				{ id: 'Cafe%20and%20Club/PaliVillageCafé.webp', label: 'Pali Village Café', video: paliVid, folder: 'Pali Village Cafe' },
				{ id: 'Cafe%20and%20Club/Taj lounges .webp', label: 'Taj Lounges', video: tajVid, folder: 'Taj Lounges' },
				{ id: 'Cafe%20and%20Club/Toto\'s.webp', label: 'Toto\'s', video: totosVid, folder: 'Toto_s' },
				{ id: 'Cafe%20and%20Club/Veronicas Pali Hill Cafes.webp', label: 'Veronicas Pali Hill Cafes', video: veronicasVid, folder: 'Veronica_s-Pali Hill Cafes' },
				{ id: 'Cafe%20and%20Club/WillingdonClub .webp', label: 'Willingdon Club', video: willingdonVid, folder: 'Willingdon club' }
			]
		},
		{
			id: 'commercial',
			name: 'Commercial',
			folder: 'Commercial',
			items: [
				{ id: 'Commercial/BKC  .webp', label: 'BKC', folder: 'BKC' },
				{ id: 'Commercial/JioWorldCentre.webp', label: 'Jio World Centre', folder: 'Jio World Centre' },
				{ id: 'Commercial/LowerParel.webp', label: 'Lower Parel', folder: 'Lower Parel' },
				{ id: 'Commercial/NMAC .webp', label: 'NMAC', folder: 'NMAC', videoName: 'NMACC (1)' },
				{ id: 'Commercial/Worli.webp', label: 'Worli', folder: 'Worli' }
			]
		},
		{
			id: 'hospital',
			name: 'Hospitals',
			folder: 'Hospitals',
			items: [
				{ id: 'Hospital/Asian Heart Institute.webp', label: 'Asian Heart Institute', folder: 'Asian Heart Institute' },
				{ id: 'Hospital/HolyFamily  .webp', label: 'Holy Family', folder: 'Holy Family Hospital' },
				{ id: 'Hospital/LilavatiHospital.webp', label: 'Lilavati Hospital', folder: 'Lilavati Hospital', videoName: 'Lilavati Hospital & Research Centre' }
			]
		},
		{
			id: 'education',
			name: 'Education Institutes',
			folder: 'Education%20Institutes',
			items: [
				{ id: 'Education/AmericanSchoolofBombay.webp', label: 'American School of Bombay', folder: 'American School of Bombay' },
				{ id: 'Education/DhirubhaiAmbani.webp', label: 'Dhirubhai Ambani', folder: 'Dhirubhai Ambani International School' },
				{ id: 'Education/StAndrews .webp', label: 'St. Andrews', folder: 'St. Andrews College', videoName: 'St. Andrew_s College' },
				{ id: 'Education/StStanislaus.webp', label: 'St. Stanislaus', folder: 'St. Stanislaus International School', videoName: 'St. Stanislaus High School' }
			]
		},
		{
			id: 'faith-heritage',
			name: 'Faith & Heritage',
			folder: 'Faith%20%26%20Heritage',
			items: [
				{ id: 'Faith%20And%20Heritage/Mount Mary Basilica.webp', label: 'Mount Mary Basilica', folder: 'Mount Mary', videoName: 'Mount Mary Basilica' }
			]
		},
		{
			id: 'retail-lifestyle',
			name: 'Retail & Lifestyle',
			folder: 'Retail%20%26%20Lifestyle',
			items: [
				{ id: 'Retail%20And%20Lifestyle/Bandstand.webp', label: 'Bandstand', folder: 'Bandstand' },
				{ id: 'Retail%20And%20Lifestyle/CarterRoad.webp', label: 'Carter Road', folder: 'Carter Road' },
				{ id: 'Retail%20And%20Lifestyle/JioWorldDrive.webp', label: 'Jio World Drive', folder: 'Jio World Drive' },
				{ id: 'Retail%20And%20Lifestyle/LinkingHillRoad .webp', label: 'Linking Hill Road', folder: 'Linking-Hill Road', videoName: 'Linking Hill Road' }
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

<!-- Right-side gallery thumbnails -->
{#if activeFolder && activeCategoryFolder}
	<div class="fixed right-6 top-1/2 -translate-y-1/2 z-[1000] flex flex-col gap-4 bg-black/40 backdrop-blur-xl border border-white/10 p-4 rounded-2xl shadow-2xl animate-fade-in select-none">
		<span class="text-[9px] tracking-widest text-[#dead66] uppercase text-center font-semibold mb-1">Gallery</span>
		{#each [1, 2, 3] as imgNum}
			{@const imgUrl = `https://assets.vestate.io/kl-rahega/images/${activeCategoryFolder}/${encodeURIComponent(activeFolder)}/${imgNum}.png`}
			<button
				class="group relative w-28 h-20 rounded-xl overflow-hidden border border-white/15 cursor-pointer shadow-md hover:border-[#dead66] transition-all duration-300 transform hover:scale-105 active:scale-95 p-0 bg-transparent"
				on:click={() => openLightbox(imgNum)}
				type="button"
			>
				<img
					src={imgUrl}
					alt="Gallery preview"
					class="w-full h-full object-cover group-hover:scale-110 transition-transform duration-500"
				/>
				<div class="absolute inset-0 bg-black/10 group-hover:bg-transparent transition-colors duration-300"></div>
			</button>
		{/each}
	</div>
{/if}

<!-- Center Lightbox Overlay -->
{#if lightboxOpen && activeFolder && activeCategoryFolder}
	<div class="fixed inset-0 bg-black/85 backdrop-blur-md z-[10000] flex items-center justify-center animate-fade-in" on:click={closeLightbox}>
		<div class="bg-black/80 border border-white/15 p-6 rounded-3xl max-w-4xl w-[90%] max-h-[85vh] flex flex-col relative shadow-2xl" on:click|stopPropagation>
			<!-- Close button -->
			<button
				class="absolute top-5 right-5 text-white/50 hover:text-white cursor-pointer transition-colors p-2 rounded-full hover:bg-white/10"
				on:click={closeLightbox}
				type="button"
			>
				<svg class="w-5 h-5" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round">
					<line x1="18" y1="6" x2="6" y2="18"></line>
					<line x1="6" y1="6" x2="18" y2="18"></line>
				</svg>
			</button>

			<!-- Title -->
			<div class="text-center mb-4">
				<h3 class="text-white text-xl font-semibold tracking-wider uppercase font-serif" style="font-family: 'The Seasons', serif;">
					{activeLabel}
				</h3>
				<p class="text-white/40 text-xs tracking-widest uppercase mt-1">Image {lightboxIndex} of 3</p>
			</div>

			<!-- Main Image & Navigation -->
			<div class="flex-1 min-h-0 flex items-center justify-between gap-4 py-2">
				<!-- Left Arrow -->
				<button
					class="text-white/50 hover:text-white cursor-pointer transition-all p-3 rounded-full hover:bg-white/10"
					on:click={prevImage}
					type="button"
				>
					<svg class="w-6 h-6" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="3" stroke-linecap="round" stroke-linejoin="round">
						<polyline points="15 18 9 12 15 6"></polyline>
					</svg>
				</button>

				<!-- Image -->
				<div class="flex-1 h-[50vh] flex items-center justify-center overflow-hidden rounded-xl bg-black/30 border border-white/5">
					<img
						src={`https://assets.vestate.io/kl-rahega/images/${activeCategoryFolder}/${encodeURIComponent(activeFolder)}/${lightboxIndex}.png`}
						alt={`${activeLabel} Gallery`}
						class="max-h-full max-w-full object-contain select-none"
					/>
				</div>

				<!-- Right Arrow -->
				<button
					class="text-white/50 hover:text-white cursor-pointer transition-all p-3 rounded-full hover:bg-white/10"
					on:click={nextImage}
					type="button"
				>
					<svg class="w-6 h-6" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="3" stroke-linecap="round" stroke-linejoin="round">
						<polyline points="9 18 15 12 9 6"></polyline>
					</svg>
				</button>
			</div>

			<!-- Mini selector thumbnails below -->
			<div class="flex justify-center gap-3 mt-6">
				{#each [1, 2, 3] as imgNum}
					<button
						class="w-20 h-14 rounded-lg overflow-hidden border transition-all duration-300 p-0 bg-transparent cursor-pointer {lightboxIndex === imgNum ? 'border-[#dead66] scale-105 shadow-md' : 'border-white/10 hover:border-white/40 opacity-60 hover:opacity-100'}"
						on:click={() => lightboxIndex = imgNum}
						type="button"
					>
						<img
							src={`https://assets.vestate.io/kl-rahega/images/${activeCategoryFolder}/${encodeURIComponent(activeFolder)}/${imgNum}.png`}
							alt="Gallery nav thumbnail"
							class="w-full h-full object-cover"
						/>
					</button>
				{/each}
			</div>
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
