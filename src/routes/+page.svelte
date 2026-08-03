<script>
	import Home from './Home.svelte';
	import { setContext, getContext } from 'svelte';
	import { writable } from 'svelte/store';
	import { goto } from '$app/navigation';
	import 'iconify-icon';
	import poweredByVretail from '$lib/images/powered-vretail.png';
	import instructionIcon from '$lib/images/instruction-icon.svg';
	import instructionPanoIcon from '$lib/images/instruction-pano.svg';
	import { onMount } from 'svelte';
	// Create a store and update it when necessary...
	const hotspotName = getContext('hotspotName');
	const currentUI = getContext('currentUI');
	const walkthroughDisabled = getContext('walkthroughDisabled');

	const UIPanel = getContext('UIPanel');

	const instructionPano = writable();
	$effect(() => {
		instructionPano.set(true);
	});

	function inIframe() {
		try {
			return window.self !== window.top;
		} catch (e) {
			return true;
		}
	}
	let isIframe = $state(inIframe());
	let introVideoMuted = $state(true);
	let isTransitioning = $state(false);
	let showParallax = $state(false);
	let hasScrolled = $state(false);
	let foliageSlidOut = $state(false);
	let foliageSettled = $state(false);
	let hasScrolledOnce = $state(false);
	let parallaxImagesLoaded = $state(false);
	let activeHover = $state(null);

	function preloadParallaxImages() {
		const images = [
			'/building.png',
			'/bg1.png',
			'/leftfull1.png',
			'/rightfull1.png',
			'/mousewheel.png'
		];
		let loadedCount = 0;
		images.forEach(src => {
			const img = new Image();
			img.onload = () => {
				loadedCount++;
				if (loadedCount === images.length) {
					parallaxImagesLoaded = true;
				}
			};
			img.onerror = () => {
				loadedCount++;
				if (loadedCount === images.length) {
					parallaxImagesLoaded = true;
				}
			};
			img.src = src;
		});
	}

	function startExperience() {
		// Request fullscreen immediately under direct user gesture
		if (!(window.self !== window.top) && window.innerWidth < 1200) {
			if (document.body.requestFullscreen) {
				document.body.requestFullscreen();
			} else if (document.body.webkitRequestFullscreen) {
				document.body.webkitRequestFullscreen();
			} else if (document.body.msRequestFullscreen) {
				document.body.msRequestFullscreen();
			}
		} 

		isTransitioning = true;
		const startTime = Date.now();
		const minLoaderTime = 2000;

		function checkReady() {
			const elapsed = Date.now() - startTime;
			if (parallaxImagesLoaded && elapsed >= minLoaderTime) {
				isTransitioning = false;
				$UIPanel = 'loaded';
				showParallax = true;
				setTimeout(() => {
					foliageSlidOut = true;
					setTimeout(() => {
						foliageSettled = true;
					}, 3000);
				}, 50);
			} else {
				setTimeout(checkReady, 100);
			}
		}

		checkReady();
	}

	$effect(() => {
		if (hasScrolled) {
			setTimeout(() => {
				goto('/menu');
			}, 2000);
		}
	});
	let touchStartY = 0;
	let accumulatedDelta = 0;
	const SCROLL_THRESHOLD = 40;

	function handleWheel(e) {
		accumulatedDelta += e.deltaY;
		if (accumulatedDelta > SCROLL_THRESHOLD) {
			hasScrolled = true;
			hasScrolledOnce = true;
			accumulatedDelta = SCROLL_THRESHOLD; // Cap it
		} else if (accumulatedDelta < -SCROLL_THRESHOLD) {
			hasScrolled = false;
			accumulatedDelta = -SCROLL_THRESHOLD; // Cap it
		}
	}

	function handleTouchStart(e) {
		touchStartY = e.touches[0].clientY;
	}

	function handleTouchMove(e) {
		const touchEndY = e.touches[0].clientY;
		const diffY = touchStartY - touchEndY; // positive = swipe up, negative = swipe down
		if (Math.abs(diffY) > SCROLL_THRESHOLD) {
			if (diffY > 0) {
				hasScrolled = true;
				hasScrolledOnce = true;
			} else {
				hasScrolled = false;
			}
			touchStartY = touchEndY; // reset anchor to prevent continuous triggering on a single swipe
		}
	}

	function handleExploreClick() {
		if (currentSlide === 1) {
			$currentUI = {
				overview: false,
				views: true,
				Exterior: false,
				interiors: false,
				amenities: false,
				highlights: false,
				vicinity: false
			};
			goto('/views');
		} else {
			$currentUI = {
				overview: false,
				views: false,
				Exterior: false,
				interiors: false,
				amenities: false,
				highlights: false,
				vicinity: true
			};
			goto('/vicinities');
		}
	}

	const toggleIntroAudio = (event) => {
		if (event) event.stopPropagation();
		const video = document.querySelector('.intro-video');
		if (video) {
			video.muted = !video.muted;
			introVideoMuted = video.muted;
			if (!video.muted) {
				video.play().catch((err) => console.log('Audio playback permission error:', err));
			}
		}
	};

	onMount(async () => {
		function setElementHeight() {
			const vh = window.innerHeight * 0.01;
			document.documentElement.style.setProperty('--vh', `${vh}px`);
		}
		setElementHeight();
		// Update the height on resize
		window.addEventListener('resize', setElementHeight);
		let lastClickTime = 0;
		const clickTimeout = 300; // Timeout in milliseconds (adjustable)
		if (!(window.self !== window.top) && window.innerWidth < 1200) {
			window.addEventListener('click', () => {
				const now = Date.now();
				if (now - lastClickTime < clickTimeout) {
					if (document.body.requestFullscreen) {
						document.body.requestFullscreen();
					} else if (document.body.webkitRequestFullscreen) {
						/* Safari */
						document.body.webkitRequestFullscreen();
					} else if (document.body.msRequestFullscreen) {
						/* IE11 */
						document.body.msRequestFullscreen();
					}
				}
				lastClickTime = now;
			});
		}

		localStorage.getItem('instructions-view-count') == 4 && instructionPano.set(false);

		preloadParallaxImages();

		const video = document.querySelector('.intro-video');
		if (video) {
			video.muted = true;
			introVideoMuted = true;
			video.play().catch((e) => console.error('Video playback failed completely:', e));
		}
	});
	setContext('currentUI', currentUI);

	const show = (ui) => {
		$currentUI = {
			interiors: false,
			amenities: false,
			interiorsDollhouse: false,
			highlights: false
		};
		$currentUI[ui] = true;
		console.log('second', $currentUI);
	};
	hotspotName.subscribe((changedHotspot) => {
		if (changedHotspot == 'Exterior') {
			$currentUI['highlights'] = false;

			show('Exterior');
		}

		if (changedHotspot == 'lawn') {
			$currentUI['highlights'] = false;

			console.log($currentUI);
			show('amenities');
		}
	});

	//
</script>

<svelte:head>
	<title>K Raheja</title>
	<meta name="description" content="kl-raheja" />
</svelte:head>

<Home />
<div class="rotate-to-view">
	<svg
		width="594"
		height="641"
		viewBox="0 0 594 641"
		fill="none"
		xmlns="http://www.w3.org/2000/svg"
	>
		<g clip-path="url(#clip0_323_2301)">
			<path d="M251.999 151H341.999" stroke="#fff" stroke-width="12" stroke-linecap="round" />
			<circle cx="296.999" cy="457" r="12" fill="#fff" />
		</g>
		<rect x="203.999" y="147" width="186" height="352" rx="12" stroke="#fff" stroke-width="8" />
		<path
			d="M343.449 618.795C341.64 617.527 341.2 615.033 342.467 613.224L363.116 583.734C364.383 581.925 366.877 581.485 368.687 582.752C370.496 584.019 370.936 586.513 369.669 588.323L351.314 614.536L377.527 632.89C379.337 634.157 379.777 636.651 378.51 638.461C377.243 640.271 374.748 640.71 372.939 639.443L343.449 618.795ZM581.096 274.468C580.713 272.292 582.165 270.218 584.341 269.834C586.517 269.451 588.591 270.903 588.975 273.079L581.096 274.468ZM345.049 611.579C421.054 598.177 488.623 555.131 532.891 491.91L539.444 496.499C493.959 561.458 424.533 605.687 346.438 619.457L345.049 611.579ZM532.891 491.91C577.158 428.69 594.498 350.474 581.096 274.468L588.975 273.079C602.745 351.174 584.928 431.54 539.444 496.499L532.891 491.91Z"
			fill="white"
		/>
		<path
			d="M349.021 32.1887C350.288 33.9983 349.848 36.4925 348.038 37.7596L318.549 58.4083C316.739 59.6754 314.245 59.2356 312.978 57.426C311.711 55.6164 312.151 53.1222 313.96 51.8551L340.173 33.5007L321.819 7.28779C320.552 5.47817 320.991 2.98399 322.801 1.71688C324.611 0.449769 327.105 0.88956 328.372 2.69918L349.021 32.1887ZM125.313 83.3514L127.607 86.628L125.313 83.3514ZM7.93895 274.47C7.55531 276.645 5.4807 278.098 3.30513 277.714C1.12953 277.331 -0.323135 275.256 0.0604705 273.08L7.93895 274.47ZM345.05 38.4222C269.044 25.0204 190.828 42.3605 127.607 86.628L123.019 80.0748C187.977 34.5903 268.344 16.7735 346.439 30.5437L345.05 38.4222ZM127.607 86.628C64.3868 130.895 21.3408 198.464 7.93895 274.47L0.0604705 273.08C13.8308 194.985 58.0601 125.559 123.019 80.0748L127.607 86.628Z"
			fill="white"
		/>
		<defs>
			<clipPath id="clip0_323_2301">
				<rect x="203.999" y="147" width="186" height="352" rx="12" fill="white" />
			</clipPath>
		</defs>
	</svg>
	Please rotate your device for better experience
</div>

<!-- <img src={lntLogo} alt="" id="lntlogo" class="absolute left-5 top-5 z-[2000000002]" /> -->
{#if $UIPanel == 'loading'}
	<div
		class="fixed left-0 top-0 z-[2000000000] h-screen w-screen rounded bg-black bg-cover bg-center bg-no-repeat"
	>
		<div class="z-[2000000002] h-screen w-screen bg-gradient-to-t from-black">
			<video
				class="intro-video"
				src="https://assets.vestate.io/webtool/kraheja/kraheja/Logos/output.mp4"
				autoplay
				muted
				loop
				playsinline
				style="width: 100vw; height: 100vh; object-fit: cover; position: absolute; left: 0; top: 0; z-index: -1;"
			></video>
			<button
				on:click={toggleIntroAudio}
				class="fixed top-5 left-5 z-[2000000003] p-3 rounded-full px-6  text-white border border-white/20 backdrop-blur-sm cursor-pointer transition-all duration-300 flex items-center justify-center"
				aria-label={introVideoMuted ? "Unmute audio" : "Mute audio"}
			>
				{#if introVideoMuted}
					<!-- Muted Speaker Icon -->
					<svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
						<polygon points="11 5 6 9 2 9 2 15 6 15 11 19 11 5"></polygon>
						<line x1="23" y1="9" x2="17" y2="15"></line>
						<line x1="17" y1="9" x2="23" y2="15"></line>
					</svg>
				{:else}
					<!-- Playing Speaker Icon -->
					<svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
						<polygon points="11 5 6 9 2 9 2 15 6 15 11 19 11 5"></polygon>
						<path d="M19.07 4.93a10 10 0 0 1 0 14.14M15.54 8.46a5 5 0 0 1 0 7.07"></path>
					</svg>
				{/if}
			</button>
			<div class="intro-container fixed inset-0 z-[2000000003] pointer-events-none select-none">
				<!-- Left Bottom Text Section -->
				<div class="intro-text-container absolute left-6 md:left-16 bottom-6 md:bottom-16 flex flex-col items-start text-left text-white max-w-[55%] md:max-w-[480px]">
					<!-- STEP › INTO -->
					<div class="flex items-center gap-2 text-[10px] md:text-sm font-light tracking-[0.3em] uppercase text-white/90">
						<span style="font-family: 'Viaoda Libre', serif !important;">STEP</span>
						<span class="text-[#c5a880] text-sm md:text-lg font-normal leading-none" style="transform: translateY(-1px); font-family: 'Viaoda Libre', serif !important;">›</span>
						<span style="font-family: 'Viaoda Libre', serif !important;">INTO</span>
					</div>

					<!-- K RAHEJA -->
					<h1 class="text-2xl md:text-5xl tracking-[0.12em] uppercase text-white my-1 md:my-3 font-normal font-serif leading-none" style="font-family: 'Viaoda Libre', serif;">
						K RAHEJA
					</h1>

					<!-- Description -->
					<p class="text-[9px] md:text-xs font-light text-white/70 leading-relaxed tracking-wider normal-case" style="font-family: 'Imprima', sans-serif;">
						Discover a location surrounded by South Mumbai's finest landmarks, cultural destinations, and lifestyle experiences, all thoughtfully connected to your everyday life.
					</p>
				</div>

				<!-- Right Bottom Button Container -->
				<div class="intro-btn-container absolute right-6 md:right-16 bottom-6 md:bottom-16 pointer-events-auto">
					<button
						id="v-start-btn"
						on:click={startExperience}
						class="get-started-btn cursor-pointer"
					>
						DISCOVER
					</button>
				</div>
			</div>
		</div>
	</div>
{/if}
{#if $UIPanel == 'instructions-exterior' && $currentUI.Exterior && $walkthroughDisabled}
	<div class="centered-panel instruction-screen p-4">
		<div class="left-panel--header mb-3 flex justify-between">
			<div class="left-title flex flex-col gap-1 text-left">
				<div class="text-2xl font-bold">Instructions</div>
				See how to explore K Raheja
			</div>
			<button
				on:click={() => {
					UIPanel.set('loaded');
					!isIframe && localStorage.setItem('instructions-view-count', '4');
				}}
				id="instruction-close"
				class="ghost-btn close-btn !px-0 !pt-0"
			>
				<svg
					width="42"
					height="42"
					viewBox="0 0 42 42"
					fill="none"
					xmlns="http://www.w3.org/2000/svg"
				>
					<rect
						x="0.363281"
						y="0.319336"
						width="41.5884"
						height="41.5884"
						rx="9.50592"
						fill="#ECECEC"
					/>
					<path
						d="M15.9062 15.8613L26.4104 26.3655"
						stroke="black"
						stroke-width="2.06369"
						stroke-linecap="round"
					/>
					<path
						d="M26.4102 15.8613L15.906 26.3655"
						stroke="black"
						stroke-width="2.06369"
						stroke-linecap="round"
					/>
				</svg>
			</button>
		</div>
		<div class="mb-4 mt-3 grid grid-cols-3 gap-2">
			<div class="col-span-1">
				Navigate and explore around with your mouse. Click on a touchpoint to interact
			</div>
			<div class="col-span-1"><img src={instructionIcon} alt="" /></div>
			<div class="col-span-1">
				<div class="mt-[2.75rem]">Use your mouse scroller to zoom an object.</div>
			</div>
		</div>
		<button
			on:click={() => {
				UIPanel.set('instructions-nav');
				!isIframe && localStorage.setItem('instructions-view-count', '1');
			}}
			class="primary-btn mx-auto block w-80"
			id="contact-submit">Okay</button
		>
	</div>
{/if}
{#if $instructionPano && $hotspotName != 'Exterior' && $hotspotName != 'ExteriorImg' && $walkthroughDisabled}
	<div class="centered-panel instruction-screen p-4">
		<div class="left-panel--header mb-3 flex justify-between">
			<div class="left-title flex flex-col gap-1 text-left">
				<div class="text-2xl font-bold">Instructions</div>
				See how to explore K Raheja
			</div>
			<button
				on:click={() => {
					instructionPano.set(false);
					!isIframe && localStorage.setItem('instructions-view-count', '4');
				}}
				id="instruction-2-close"
				class="ghost-btn close-btn !px-0 !pt-0"
			>
				<svg
					width="42"
					height="42"
					viewBox="0 0 42 42"
					fill="none"
					xmlns="http://www.w3.org/2000/svg"
				>
					<rect
						x="0.363281"
						y="0.319336"
						width="41.5884"
						height="41.5884"
						rx="9.50592"
						fill="#ECECEC"
					/>
					<path
						d="M15.9062 15.8613L26.4104 26.3655"
						stroke="black"
						stroke-width="2.06369"
						stroke-linecap="round"
					/>
					<path
						d="M26.4102 15.8613L15.906 26.3655"
						stroke="black"
						stroke-width="2.06369"
						stroke-linecap="round"
					/>
				</svg>
			</button>
		</div>
		<div class="mb-4 mt-3 grid grid-cols-2 gap-2">
			<div class="col-span-1"><img src={instructionPanoIcon} alt="" /></div>
			<div class="col-span-1">
				<div class="mt-[2.75rem]">
					Click on the arrows highlighted on the floor to navigate, Rotate left & right with you
					mouse to view the surrounding.
				</div>
			</div>
		</div>
		<button
			on:click={() => {
				instructionPano.set(false);
				!isIframe && localStorage.setItem('instructions-view-count', '4');
			}}
			class="primary-btn mx-auto block w-80"
			id="contact-submit">Okay</button
		>
	</div>
{/if}

{#if isTransitioning}
	<div
		class="secondary-loading-screen fixed left-0 top-0 z-[2000000010] h-screen w-screen flex flex-col items-center justify-center bg-cover bg-center bg-no-repeat"
		style="background-image: url('/blurbg.png');"
	>
		<img class="loading-building-img mb-6 max-h-[150px] w-auto" src="Vector.png" alt="" />
		<div class="loading-text text-xl md:text-2xl font-light tracking-[0.25em] text-[#e5d5be]" style="font-family: 'The Seasons', serif;">
			LOADING<span class="dot-anim"></span>
		</div>
	</div>
{/if}

{#if showParallax}
	<div
		on:wheel={handleWheel}
		on:touchstart={handleTouchStart}
		on:touchmove={handleTouchMove}
		class="parallax-scroll-container fixed left-0 top-0 z-[2000000020] h-screen w-screen bg-black overflow-hidden select-none"
		class:has-scrolled-once={hasScrolledOnce}
	>
		<div class="h-full w-full relative">
			{#if foliageSettled && !hasScrolled}
				<div class="absolute inset-0 z-[5] flex pointer-events-auto">
					<div
						class="w-[33vw] h-full cursor-default"
						on:mouseenter={() => activeHover = 'left'}
						on:mouseleave={() => activeHover = null}
					></div>
					<div
						class="w-[34vw] h-full cursor-default"
						on:mouseenter={() => activeHover = 'center'}
						on:mouseleave={() => activeHover = null}
					></div>
					<div
						class="w-[33vw] h-full cursor-default"
						on:mouseenter={() => activeHover = 'right'}
						on:mouseleave={() => activeHover = null}
					></div>
				</div>
			{/if}

			<!-- Background building image (z-index: 1) -->
			<div
				class="fixed inset-0 pointer-events-none flex items-center justify-center parallax-bg-container"
				style="transition: transform {hasScrolled ? '4.0s ease-in-out' : '2.0s ease-out'}; transform: scale({hasScrolled ? 1.1 : 1.0}) translateY({hasScrolled ? '-40px' : '0px'}); z-index: 1;"
			>
				<img class="w-full h-full object-cover" src="/building.png" alt="Building background" />
			</div>

			<!-- Middle aperture mask image (z-index: 2) -->
			<div
				class="fixed inset-0 pointer-events-none flex items-center justify-center parallax-mask-container"
				style="
					transition: transform {hasScrolled ? '4.0s ease-in-out' : foliageSettled ? '0.8s cubic-bezier(0.25, 1, 0.5, 1)' : '2.0s ease-out'}, opacity {hasScrolled ? '3.5s ease-in-out' : '1.8s ease-out'};
					transform: scale({hasScrolled ? 7.0 : activeHover === 'center' ? 0.72 : 0.7}) translateX({hasScrolled ? '0px' : activeHover === 'left' ? '30px' : activeHover === 'right' ? '-30px' : '0px'});
					opacity: {hasScrolled ? 0 : 1};
					z-index: 2;
				"
			>
				<img
					class="w-full h-full object-cover transition-transform duration-500 scale-150"
					src="/bg1.png"
					alt="Aperture mask layer"
				/>
			</div>

			<!-- Left side floral PNG (z-index: 3) -->
			<div
				class="foliage-container left-foliage {foliageSettled ? 'settled' : ''}"
				class:scrolled={hasScrolled}
				class:slid-out={foliageSlidOut && !hasScrolled}
				style="left: 0; {foliageSettled && !hasScrolled ? `transform: translateX(${activeHover === 'left' ? '-32%' : activeHover === 'center' || activeHover === 'right' ? '-38%' : '-35%'}) scale(${activeHover === 'left' ? 1.02 : activeHover === 'center' || activeHover === 'right' ? 0.98 : 1.0}); transition: transform 0.8s cubic-bezier(0.25, 1, 0.5, 1);` : ''}"
			>
				<img class="w-full h-full object-cover object-right" src="/leftfull1.png" alt="Floral frame left" />
			</div>

			<!-- Right side floral PNG (z-index: 3) -->
			<div
				class="foliage-container right-foliage {foliageSettled ? 'settled' : ''}"
				class:scrolled={hasScrolled}
				class:slid-out={foliageSlidOut && !hasScrolled}
				style="right: 0; {foliageSettled && !hasScrolled ? `transform: translateX(${activeHover === 'right' ? '32%' : activeHover === 'center' || activeHover === 'left' ? '38%' : '35%'}) scale(${activeHover === 'right' ? 1.02 : activeHover === 'center' || activeHover === 'left' ? 0.98 : 1.0}); transition: transform 0.8s cubic-bezier(0.25, 1, 0.5, 1);` : ''}"
			>
				<img class="w-full h-full object-cover object-left" src="/rightfull1.png" alt="Floral frame right" />
			</div>

			<!-- Bottom Dark Overlay Gradient for Scroll Badge -->
			<div
				class="fixed bottom-0 left-0 w-full h-[15rem]  bg-gradient-to-t z-10 from-black/85 via-black/40 to-transparent pointer-events-none transition-opacity duration-500"
				style="opacity: {hasScrolled ? 0 : 1};"
			></div>

			<!-- Scroll Indicator Badge -->
			<button
				on:click={() => hasScrolled = true}
				class="fixed bottom-4 left-1/2 -translate-x-1/2 brightness-150 cursor-pointer flex flex-col items-center z-10 transition-opacity duration-500 bg-transparent border-0 p-0 outline-none"
				style="opacity: {hasScrolled ? 0 : 1}; pointer-events: {hasScrolled ? 'none' : 'auto'};"
			>
				<img 
					class="w-[80%] h-[80%] object-cover " 
					src="/mousewheel.png"
					alt="Scroll to experience"
				/>
			</button>


			<!-- Fullscreen Transition Blur Overlay -->
			<div class="scroll-transition-overlay" class:active={hasScrolled}></div>
		</div>
	</div>
{/if}

<style>
	button {
		background-size: 100% 100%;
		background-repeat: no-repeat;
		padding: 1rem 0.5rem;
		font-size: 1rem;
		border: 0;
	}

	.get-started-btn {
		background-color: rgba(30, 30, 30, 0.45) !important;
		border: 1.5px solid #c5a880 !important;
		border-radius: 9999px !important;
		color: #e5d5be !important;
		padding: 0.75rem 2.5rem !important;
		font-size: 1.05rem !important;
		letter-spacing: 0.12em !important;
		text-transform: none !important;
		font-weight: normal !important;
		transition: all 0.4s ease-in-out !important;
		cursor: pointer !important;
		box-shadow: 0 4px 20px rgba(0, 0, 0, 0.3) !important;
		font-family: 'The Seasons', serif !important;
		display: inline-flex !important;
		align-items: center !important;
		justify-content: center !important;
		height: auto !important;
		width: auto !important;
		min-width: 160px !important;
		animation: none !important;
	}

	.get-started-btn:hover {
		background-color: #DEAD66 !important; 
		font-weight: bold !important;
		color: #ffffff !important;
		border-color: #DEAD66 !important;
		transform: scale(1.03) !important;
		animation: none !important;
	}

	@media (max-width: 768px), (max-width: 950px) and (orientation: landscape) {
		.get-started-btn {
			padding: 0.5rem 1.75rem !important;
			font-size: 0.8rem !important;
			min-width: 120px !important;
		}
	}

	.centered-panel {
		border-radius: 0.8rem;
		position: absolute;
		bottom: 2rem;
		z-index: 99;
		transform-origin: center;
		top: 0;
		left: 0;
		width: 500px;
		height: fit-content;
		right: 0;
		bottom: 0;
		margin: auto;
		background-color: #fff;
		flex-wrap: wrap;
	}
	.instruction-screen {
		width: 600px;
	}

	/* Loader Animation */
	.loader {
		position: relative;
		width: 80px;
		height: 80px;
	}

	.loader-ring {
		position: absolute;
		width: 100%;
		height: 100%;
		border: 4px solid transparent;
		border-top-color: #ffd400;
		border-radius: 50%;
		animation: spin 1.5s cubic-bezier(0.68, -0.55, 0.265, 1.55) infinite;
	}

	.loader-ring:nth-child(1) {
		animation-delay: -0.45s;
	}

	.loader-ring:nth-child(2) {
		animation-delay: -0.3s;
	}

	.loader-ring:nth-child(3) {
		animation-delay: -0.15s;
	}

	@keyframes spin {
		0% {
			transform: rotate(0deg) scale(0.5);
		}
		50% {
			transform: rotate(180deg) scale(0.8);
		}
		100% {
			transform: rotate(360deg) scale(0.5);
		}
	}

	.dot-anim {
		display: inline-block;
		width: 1.5em;
		text-align: left;
	}

	.dot-anim::after {
		content: '';
		animation: dots 1.5s steps(4, end) infinite !important;
	}

	@keyframes dots {
		0%, 20% { content: ''; }
		40% { content: '.'; }
		60% { content: '..'; }
		80%, 100% { content: '...'; }
	}

	@keyframes mouse-wheel {
		0% {
			transform: translateY(-2px);
			opacity: 0;
		}
		50% {
			opacity: 1;
		}
		100% {
			transform: translateY(4px);
			opacity: 0;
		}
	}
	
	.mouse-wheel-anim {
		animation: mouse-wheel 1.6s infinite ease-in-out;
		transform-origin: center;
	}

	.parallax-bg-container {
		will-change: transform;
		backface-visibility: hidden;
		-webkit-backface-visibility: hidden;
	}

	.parallax-mask-container {
		will-change: transform, opacity;
		backface-visibility: hidden;
		-webkit-backface-visibility: hidden;
	}

	.foliage-container {
		position: fixed;
		top: 0;
		bottom: 0;
		width: 55vw;
		z-index: 3;
		pointer-events: none;
		transform: translateX(0%);
		will-change: transform;
		backface-visibility: hidden;
		-webkit-backface-visibility: hidden;
	}

	/* Left Foliage States */
	.left-foliage.slid-out {
		transform: translateX(-35%);
		transition: transform 3.0s cubic-bezier(0.16, 1, 0.3, 1);
		transition-delay: 0s;
	}

	.has-scrolled-once .left-foliage.slid-out {
		transition-delay: 0.8s; /* Wait for mask to return first */
	}

	.left-foliage.scrolled {
		transform: translateX(-105%);
		transition: transform 2.5s cubic-bezier(0.16, 1, 0.3, 1);
		transition-delay: 0s;
	}

	/* Right Foliage States */
	.right-foliage.slid-out {
		transform: translateX(35%);
		transition: transform 3.0s cubic-bezier(0.16, 1, 0.3, 1);
		transition-delay: 0s;
	}

	.has-scrolled-once .right-foliage.slid-out {
		transition-delay: 0.8s; /* Wait for mask to return first */
	}

	.right-foliage.scrolled {
		transform: translateX(105%);
		transition: transform 2.5s cubic-bezier(0.16, 1, 0.3, 1);
		transition-delay: 0s;
	}

	/* Initial load state (before slided out) */
	.foliage-container:not(.slid-out):not(.scrolled) {
		transform: translateX(0%);
		transition: transform 3.0s cubic-bezier(0.16, 1, 0.3, 1);
		transition-delay: 0s;
	}

	/* Hover States (Settle Phase) */
	.foliage-container.settled {
		pointer-events: auto !important;
	}

	/* .left-foliage.settled:hover {
		transform: translateX(-31%) scale(1.02) !important;
		transition: transform 0.6s cubic-bezier(0.25, 1, 0.5, 1) !important;
		transition-delay: 0s !important;
	}

	.right-foliage.settled:hover {
		transform: translateX(31%) scale(1.02) !important;
		transition: transform 0.6s cubic-bezier(0.25, 1, 0.5, 1) !important;
		transition-delay: 0s !important;
	} */

	.foliage-container img {
		transition: transform 0.6s cubic-bezier(0.25, 1, 0.5, 1);
	}

	/* Navigation Cards Stack & Design */
.nav-card {
    position: absolute;
    inset: 0;
    border-radius: 28px;
    padding: 28px;
    display: flex;
    flex-direction: column;
    justify-content: space-between;
    color: #f3efe9;
 
     background: rgba(255, 255, 255, 0.1);
    backdrop-filter: blur(28px) saturate(1.3);
    -webkit-backdrop-filter: blur(28px) saturate(1.3);
    border: 1px solid rgba(255,255,255,.14);
    box-shadow:
      0 20px 60px rgba(0,0,0,.45),
      inset 0 1px 0 rgba(255,255,255,.08);
 
    transition: transform .55s cubic-bezier(.19,1,.22,1),
                opacity .55s ease,
                filter .55s ease;
    cursor: pointer;
  }
 
  .nav-card::before {
    content: "";
    position: absolute;
    inset: 0;
    border-radius: inherit;
   background: rgba(255, 255, 255, 0.1);

    pointer-events: none;
  }
 
  /* stacking states */
  .nav-card.front {
    z-index: 20;
    opacity: 1;
    transform: translate(0,0) scale(1);
    pointer-events: auto;
    filter: blur(0);
  }

	.nav-card.back {
   z-index: 12;
    opacity: .55;
    transform: translate(14px,-16px) scale(.95);
    pointer-events: none;
    filter: blur(1.5px);
	}

	.thumbnail-cutout {
		position: absolute;
		right: 0;
		top: 0;
		bottom: 0;
		width: 45%;
		mask: radial-gradient(circle at 100% 100%, transparent 48px, black 49px);
		-webkit-mask: radial-gradient(circle at 100% 100%, transparent 48px, black 49px);
		border-top-left-radius: 2.5rem;
		border-bottom-left-radius: 2.5rem;
		border-top-right-radius: 2.5rem;
		overflow: hidden;
	}

	.scroll-transition-overlay {
		position: fixed;
		inset: 0;
		z-index: 2000000050;
		background: rgba(10, 10, 10, 0.0);
		backdrop-filter: blur(0px);
		-webkit-backdrop-filter: blur(0px);
		pointer-events: none;
		transition: background-color 0.7s ease-in-out, backdrop-filter 0.7s ease-in-out;
		transition-delay: 1.5s;
	}

	.scroll-transition-overlay.active {
		background: rgba(10, 10, 10, 0.45);
		backdrop-filter: blur(25px);
		-webkit-backdrop-filter: blur(25px);
	}
</style>
