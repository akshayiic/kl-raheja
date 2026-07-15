<script>
	import { getContext } from 'svelte';
	import minimizeBtn from '$lib/images/minimize-icon.svg';
	import maximizeBtn from '$lib/images/maximize-icon.svg';
	import * as Accordion from '$lib/components/ui/accordion/index.ts';
	import { createLoadObserver } from '$lib/utils.ts';
	import { writable } from 'svelte/store';

	import { slide } from 'svelte/transition';
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

	import { onMount } from 'svelte';
	import { browser } from '$app/environment';
	onMount(async () => {
		if (!document.querySelector('.vicinity-box').id) {
			window.CI360.init();
		}

		console.log('sss');
	});

	// Organized vicinities from local folder structure
	const isDay = writable();

	// Vicinity data organized by category
	const vicinityCategories = [
		{
			id: 'connectivity',
			name: 'Connectivity',
			folder: 'Connectivity',
			items: [
				{ id: 'Connectivity/BandraTerminus.webp', label: 'Bandra Terminus', subcategory: 'Railway' },
				{ id: 'Connectivity/BKCMetrostation.webp', label: 'BKC Metro Station', subcategory: 'Metro' },
				{ id: 'Connectivity/SantacruzMetroLine.webp', label: 'Santa Cruz Metro Line', subcategory: 'Metro' },
				{ id: 'Connectivity/CoastalRoad.webp', label: 'Coastal Road', subcategory: 'Roadways' },
				{ id: 'Connectivity/Versova Bandra Sea Link.webp', label: 'Versova Bandra Sea Link', subcategory: 'Roadways' },
				{ id: 'Connectivity/WesternExpressHighway.webp', label: 'Western Express Highway', subcategory: 'Roadways' },
				{ id: 'Connectivity/WorliSeaLink.webp', label: 'Worli Sea Link', subcategory: 'Roadways' }
			]
		},
		{
			id: 'cafe-club',
			name: 'Cafes & Clubs',
			folder: 'Cafe%20and%20Club',
			items: [
				{ id: 'Cafe%20and%20Club/Bastian.webp', label: 'Bastian' },
				{ id: 'Cafe%20and%20Club/Khar Gymkhana.webp', label: 'Khar Gymkhana' },
				{ id: 'Cafe%20and%20Club/MIG.webp', label: 'MIG' },
				{ id: 'Cafe%20and%20Club/NationalSportsClubofIndia.webp', label: 'National Sports Club of India' },
				{ id: 'Cafe%20and%20Club/Otters.webp', label: 'Otters' },
				{ id: 'Cafe%20and%20Club/PaliVillageCafé.webp', label: 'Pali Village Café' },
				{ id: 'Cafe%20and%20Club/Taj lounges .webp', label: 'Taj Lounges' },
				{ id: 'Cafe%20and%20Club/Toto\'s.webp', label: 'Toto\'s' },
				{ id: 'Cafe%20and%20Club/Veronicas Pali Hill Cafes.webp', label: 'Veronicas Pali Hill Cafes' },
				{ id: 'Cafe%20and%20Club/WillingdonClub .webp', label: 'Willingdon Club' }
			]
		},
		{
			id: 'commercial',
			name: 'Commercial',
			folder: 'Commercial',
			items: [
				{ id: 'Commercial/BKC  .webp', label: 'BKC' },
				{ id: 'Commercial/JioWorldCentre.webp', label: 'Jio World Centre' },
				{ id: 'Commercial/LowerParel.webp', label: 'Lower Parel' },
				{ id: 'Commercial/NMAC .webp', label: 'NMAC' },
				{ id: 'Commercial/Worli.webp', label: 'Worli' }
			]
		},
		{
			id: 'hospital',
			name: 'Hospitals',
			folder: 'Hospital',
			items: [
				{ id: 'Hospital/Asian Heart Institute.webp', label: 'Asian Heart Institute' },
				{ id: 'Hospital/HolyFamily  .webp', label: 'Holy Family' },
				{ id: 'Hospital/LilavatiHospital.webp', label: 'Lilavati Hospital' }
			]
		},
		{
			id: 'education',
			name: 'Education Institutes',
			folder: 'Education',
			items: [
				{ id: 'Education/AmericanSchoolofBombay.webp', label: 'American School of Bombay' },
				{ id: 'Education/DhirubhaiAmbani.webp', label: 'Dhirubhai Ambani' },
				{ id: 'Education/StAndrews .webp', label: 'St. Andrews' },
				{ id: 'Education/StStanislaus.webp', label: 'St. Stanislaus' }
			]
		},
		{
			id: 'faith-heritage',
			name: 'Faith & Heritage',
			folder: 'Faith%20And%20Heritage',
			items: [
				{ id: 'Faith%20And%20Heritage/Mount Mary Basilica.webp', label: 'Mount Mary Basilica' }
			]
		},
		{
			id: 'retail-lifestyle',
			name: 'Retail & Lifestyle',
			folder: 'Retail%20And%20Lifestyle',
			items: [
				{ id: 'Retail%20And%20Lifestyle/Bandstand.webp', label: 'Bandstand' },
				{ id: 'Retail%20And%20Lifestyle/CarterRoad.webp', label: 'Carter Road' },
				{ id: 'Retail%20And%20Lifestyle/JioWorldDrive.webp', label: 'Jio World Drive' },
				{ id: 'Retail%20And%20Lifestyle/LinkingHillRoad .webp', label: 'Linking Hill Road' }
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

<div class={'d-block visible absolute bottom-0 left-0 right-0 top-0'}>
	<div
		class={`cloudimage-360  vicinity-box z-50 ${$isDay || $vicinityImg != '-' ? 'invisible !absolute' : 'visible !absolute'}`}
		data-folder="https://assets.vestate.io/webtool/kraheja/kraheja/vicinities/day/"
		data-filename="{'{index}.webp'}"
		data-amount="24"
		data-keys="false"
		data-filters="blur:20"
		data-drag-speed="400"
		data-request-responsive-images="true"
		data-info="false"
		data-ratio="1"
	></div>

	<!-- <div
		class={`cloudimage-360  vicinity-box z-50 ${!$isDay || $vicinityImg != '-' ? 'invisible' : 'visible  !absolute '}`}
		data-folder="https://assets.vestate.io/nahar-chandivalley/vicinities/night/"
		data-filename="{'vic{'}index{'}'}.webp"
		data-amount="24"
		data-keys="false"
		data-filters="blur:20"
		data-drag-speed="400"
		data-request-responsive-images="true"
		data-info="false"
		data-ratio="1"
	></div> -->

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
		</div>
	{/if}
</div>
