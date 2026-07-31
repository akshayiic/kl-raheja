<script>
	import { goto } from '$app/navigation';
	import { getContext, onMount } from 'svelte';
	import { flip } from 'svelte/animate';

	const currentUI = getContext('currentUI');
	const UIPanel = getContext('UIPanel');

	let currentSlide = $state(1);
	let card1W = $state(420);
	let card1H = $state(180);
	let card2W = $state(420);
	let card2H = $state(180);

	onMount(() => {
		$UIPanel = 'loaded';
		$currentUI = {
			overview: true,
			views: false,
			Exterior: false,
			interiors: false,
			amenities: false,
			highlights: false,
			vicinity: false
		};
	});

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
</script>

<div class="menu-container fixed inset-0 w-screen h-screen bg-black overflow-hidden select-none">
	<!-- Settled Parallax Background Building -->
	<div class="fixed inset-0 pointer-events-none flex items-center justify-center" style="z-index: 1;">
		<img class="w-full h-full object-cover scale-110" src="/building.png" alt="Building background" />
	</div>

	<!-- Bottom Dark Overlay Gradient -->
	<div class="fixed bottom-0 left-0 w-full h-[20rem] bg-gradient-to-t z-10 from-black/90 via-black/45 to-transparent pointer-events-none"></div>

	<!-- Navigation UI Description Overlay -->
	<div class="fixed bottom-12 left-12 max-w-[800px] z-[25] flex flex-col gap-2 pointer-events-auto text-left animate-fade-in">
		<p class="text-white/80 text-xs md:text-sm font-normal leading-relaxed tracking-wide normal-case" style="font-family: 'Imprima', sans-serif;">
			In the heart of South Mumbai, where heritage meets contemporary living, Raheja SOBO Residences presents a rare collection of thoughtfully crafted homes. An address defined by timeless architecture, exceptional views, and a neighbourhood that has shaped the city's finest lifestyles.
		</p>
	</div>

	<!-- Bottom Right Navigation Card & Controls -->
	<div class="fixed bottom-12 right-6 z-[25] flex items-end gap-10 pointer-events-auto animate-fade-in">
		<div class="relative w-[420px] h-[180px]">
			<!-- Toggle/Next circular button -->
			<button
				on:click={() => currentSlide = currentSlide === 1 ? 2 : 1}
				class="absolute -left-12 top-1/2 -translate-y-1/2 w-10 h-10 rounded-full border border-[#c5a880]/60 hover:border-[#c5a880] bg-black/40 hover:bg-black/60 transition-all duration-300 flex items-center justify-center text-[#e5d5be] hover:text-white cursor-pointer z-30 !p-0"
				aria-label="Next slide"
			>
				<svg width="8" height="8" viewBox="0 0 10 10" fill="currentColor">
					<circle cx="5" cy="2" r="1.1" />
					<circle cx="2" cy="6.8" r="1.1" />
					<circle cx="8" cy="6.8" r="1.1" />
				</svg>
			</button>

			<!-- Card 2 (VICINITY) -->
			<div 
				bind:clientWidth={card2W} 
				bind:clientHeight={card2H}
				class="nav-card {currentSlide === 2 ? 'front' : 'back'} cursor-pointer p-4"
				on:click={() => currentSlide !== 2 && (currentSlide = 2)}
			>
				<svg class="absolute inset-0 w-full h-full pointer-events-none" style="z-index: -1;">
					<path d="M 40,0 L {card2W - 40},0 A 40,40 0 0 1 {card2W},40 L {card2W},{card2H - 48} A 48,48 0 0 0 {card2W - 48},{card2H} L 40,{card2H} A 40,40 0 0 1 0,{card2H - 40} L 0,40 A 40,40 0 0 1 40,0 Z" 
					      fill="rgba(255, 255, 255, 0.08)" 
					      stroke="rgba(255, 255, 255, 0.15)" 
					      stroke-width="1.2" />
				</svg>

				<!-- Text section -->
				<div class="flex flex-col flex-1 text-left relative z-10 pr-[48%]">
					<h2 class="text-xl tracking-[0.15em] text-white mb-1.5 font-semibold uppercase" style="font-family: 'The Seasons', serif;">
						VICINITY
					</h2>
					<p class="text-white text-[13px] leading-relaxed mb-2.5 normal-case font-light max-w-[200px]" style="font-family: 'Imprima', sans-serif; letter-spacing: 0.02em;">
						Discover South Mumbai's finest landmarks, cultural destinations, and lifestyle experiences close to your home.
					</p>
					<!-- Explore button -->
					<button
						on:click={(e) => { e.stopPropagation(); handleExploreClick(); }}
						class="explore-card-btn self-start flex items-center justify-center gap-2 px-5 py-2 rounded-full border border-[#c5a880] text-[10px] text-[#e5d5be] tracking-widest bg-transparent hover:bg-[#c5a880] hover:text-[#1e1e1e] transition-all duration-300 cursor-pointer !h-auto !w-auto"
						style="font-family: 'Imprima', sans-serif;"
					>
						<svg width="8" height="8" viewBox="0 0 10 10" fill="currentColor">
							<circle cx="5" cy="2" r="1" />
							<circle cx="2" cy="7" r="1" />
							<circle cx="8" cy="7" r="1" />
						</svg>
						Explore
					</button>
				</div>
				
				<!-- Thumbnail Image -->
				<div class="thumbnail-cutout z-10">
					<img 
						class="w-full h-full object-cover pointer-events-none" 
						src="/vicinity-thumbnail.png" 
						alt="Vicinity" 
					/>
				</div>
			</div>

			<!-- Card 1 (VIEWS) -->
			<div 
				bind:clientWidth={card1W} 
				bind:clientHeight={card1H}
				class="nav-card {currentSlide === 1 ? 'front' : 'back'} cursor-pointer p-4"
				on:click={() => currentSlide !== 1 && (currentSlide = 1)}
			>
				<svg class="absolute inset-0 w-full h-full pointer-events-none" style="z-index: -1;">
					<path d="M 40,0 L {card1W - 40},0 A 40,40 0 0 1 {card1W},40 L {card1W},{card1H - 48} A 48,48 0 0 0 {card1W - 48},{card1H} L 40,{card1H} A 40,40 0 0 1 0,{card1H - 40} L 0,40 A 40,40 0 0 1 40,0 Z" 
					      fill="rgba(255, 255, 255, 0.08)" 
					      stroke="rgba(255, 255, 255, 0.15)" 
					      stroke-width="1.2" />
				</svg>

				<!-- Text section -->
				<div class="flex flex-col flex-1 text-left relative z-10 pr-[48%]">
					<h2 class="text-xl tracking-[0.15em] text-white mb-1.5 font-semibold uppercase" style="font-family: 'The Seasons', serif;">
						VIEWS
					</h2>
					<p class="text-white text-[13px] leading-relaxed mb-2.5 normal-case font-light max-w-[200px]" style="font-family: 'Imprima', sans-serif; letter-spacing: 0.02em;">
						Explore The Building From Multiple Viewpoints And Discover Every Angle Of Its Architecture And Surroundings.
					</p>
					<!-- Explore button -->
					<button
						on:click={(e) => { e.stopPropagation(); handleExploreClick(); }}
						class="explore-card-btn self-start flex items-center justify-center gap-2 px-5 py-2 rounded-full border border-[#c5a880] text-[10px] text-[#e5d5be] tracking-widest bg-transparent hover:bg-[#c5a880] hover:text-[#1e1e1e] transition-all duration-300 cursor-pointer !h-auto !w-auto"
						style="font-family: 'Imprima', sans-serif;"
					>
						<svg width="8" height="8" viewBox="0 0 10 10" fill="currentColor">
							<circle cx="5" cy="2" r="1" />
							<circle cx="2" cy="7" r="1" />
							<circle cx="8" cy="7" r="1" />
						</svg>
						Explore
					</button>
				</div>
				
				<!-- Thumbnail Image -->
				<div class="thumbnail-cutout z-10">
					<img 
						class="w-full h-full object-cover pointer-events-none" 
						src="/views-thumbnail.png" 
						alt="Views" 
					/>
				</div>
			</div>
		</div>

		<!-- Page Indicators -->
		<div class="flex items-center gap-4 mb-2 select-none" style="font-family: 'The Seasons', serif;">
			{#each currentSlide === 1 ? [1, 2] : [2, 1] as slide, index (slide)}
				<div animate:flip={{ duration: 500 }} class="indicator-item flex items-center">
					<button 
						on:click={() => currentSlide = slide}
						class="indicator-btn {index === 0 ? 'active' : ''}"
					>
						{slide < 10 ? `0${slide}` : slide}
					</button>
				</div>
			{/each}
		</div>
	</div>
</div>

<style>
	.animate-fade-in {
		animation: fadeIn 1.2s cubic-bezier(0.16, 1, 0.3, 1) forwards;
	}

	@keyframes fadeIn {
		from {
			opacity: 0;
			transform: translateY(20px);
		}
		to {
			opacity: 1;
			transform: translateY(0);
		}
	}

	button {
		background-size: 100% 100%;
		background-repeat: no-repeat;
		padding: 1rem 0.5rem;
		font-size: 1rem;
		border: 0;
	}

	.explore-card-btn {
		background-color: rgba(30, 30, 30, 0.45) !important;
		border: 1.5px solid #c5a880 !important;
		border-radius: 9999px !important;
		color: #e5d5be !important;
		padding: 0.5rem 1.5rem !important;
		font-size: 0.75rem !important;
		letter-spacing: 0.12em !important;
		text-transform: uppercase !important;
		font-weight: normal !important;
		transition: all 0.4s ease-in-out !important;
		cursor: pointer !important;
		box-shadow: 0 4px 20px rgba(0, 0, 0, 0.3) !important;
		display: inline-flex !important;
		align-items: center !important;
		justify-content: center !important;
		height: auto !important;
		width: auto !important;
	}

	.explore-card-btn:hover {
		background-color: #c5a880 !important;
		color: #1e1e1e !important;
		box-shadow: 0 0 25px rgba(197, 168, 128, 0.65) !important;
		border-color: #c5a880 !important;
	}

	/* Navigation Cards Stack & Design */
.nav-card {
	position: absolute;
	inset: 0;
	width: 100%;
	height: 110%;
	display: flex;
	align-items: center;
	padding: 1rem 2rem;
	overflow: hidden;
	border-radius: 2.5rem;

	
    background: rgba(101, 92, 90, 0.78);

    backdrop-filter:
        blur(8px)
        brightness(.75)
        saturate(.95);

    -webkit-backdrop-filter:
        blur(8px)
        brightness(.75)
        saturate(.95);

    border: 1px solid rgba(255,255,255,.05);

    box-shadow:
        0 20px 40px rgba(0,0,0,.35),
        inset 0 1px rgba(255,255,255,.05);

 

 

	transition: all .7s cubic-bezier(.25,1,.5,1);
}

	.nav-card.front {
		transform: scale(1) translate3d(0, 0, 0);
		z-index: 20;
		opacity: 1;
		filter: blur(0);
	}

	.nav-card.back {
		transform: scale(0.85) translate3d(24px, -24px, 0);
		z-index: 10;
		opacity: 0.35;
		filter: blur(4px);
	}

	/* Concentric Cutout Mask for Thumbnail on Right Side */
	.thumbnail-cutout {
		position: absolute;
		top: 0;
		bottom: 0;
		right: 0;
		width: 45%;
		background: transparent;
		border-top-right-radius: 2.5rem;
		border-bottom-right-radius: 2.5rem;
		overflow: hidden;
		-webkit-mask-image: radial-gradient(circle at 100% 100%, transparent 48px, black 49px);
		mask-image: radial-gradient(circle at 100% 100%, transparent 48px, black 49px);
	}

	.thumbnail-cutout img {
		border-top-left-radius: 2.5rem;
		border-bottom-left-radius: 2.5rem;
	}

	.indicator-item {
		display: flex;
		align-items: center;
	}

	.indicator-item:not(:first-child)::before {
		content: "";
		width: 4rem;
		height: 1.5px;
		background: rgba(255, 255, 255, 0.2);
		margin-right: 1rem;
		display: block;
	}

	.indicator-btn {
		cursor: pointer !important;
		background: transparent !important;
		border: 0 !important;
		padding: 0 !important;
		user-select: none !important;
		transition: all 0.5s ease !important;
		color: rgba(255, 255, 255, 0.3) !important;
		font-size: 14px !important;
		font-weight: 300 !important;
	}

	.indicator-btn.active {
		color: #ffffff !important;
		font-size: 24px !important;
		font-weight: 400 !important;
	}
</style>
