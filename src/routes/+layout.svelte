<script>
	import { goto } from '$app/navigation';
	import { page } from '$app/stores';
	import { onMount, setContext } from 'svelte';
	import { writable } from 'svelte/store';
	import '../app.pcss';
	import './styles.css';

	const navSlide = writable();
	setContext('navSlide', navSlide);

	const hotspotName = writable(0);
	const UIPanel = writable('loading');
	const walkthroughDisabled = writable();
	$: walkthroughDisabled.set(false);

	$: hotspotName.set('overview');
	setContext('hotspotName', hotspotName);
	setContext('UIPanel', UIPanel);

	let currentUI = writable({
		overview: false,
		Exterior: false,
		interiors: false,
		amenities: false,
		highlights: false
	});
	setContext('currentUI', currentUI);

	const show = (ui) => {
		var newUI = {
			overview: false,
			Exterior: false,
			interiors: false,
			amenities: false,
			highlights: false
		};
		newUI[ui] = true;
		currentUI.set(newUI);
		console.log('second', $currentUI);
	};
	const vicinityImg = writable();
	$: vicinityImg.set('-');
	setContext('vicinityImg', vicinityImg);

	// check for the iframe window
	function inIframe() {
		try {
			return window.self !== window.top;
		} catch (e) {
			return true;
		}
	}
	var isIframe = inIframe();
	const switchChecker = () => {
		// eslint-disable-next-line svelte/valid-compile
		switch ($page.url.searchParams.get('panel')) {
			case 'interiors':
			show('interiors');
			UIPanel.set('loaded');
			break;
			case 'contact-us':
			show('contact');
			UIPanel.set('contact');
			break;
			case 'brochure':
			show('Brochure');
			UIPanel.set('Brochure');
			break;
			case 'amenities':
			show('amenities');
			UIPanel.set('loaded');
			break;
			case 'vicinity':
			show('vicinity');
			UIPanel.set('loaded');
			break;
			case 'overview':
			show('Exterior');
			console.log('setting overview');
			UIPanel.set('loaded');
			break;
			case 'highlights':
			show('highlights');
			UIPanel.set('loaded');
			break;
			default:
			break;
		}
	};
	onMount(() => {
		switchChecker();
		console.log('page changed', $page.url);
		if ($page.url.href.includes('brochure')) {
			show('Brochure');
			UIPanel.set('Brochure');
			// CI360.
		}
		if ($page.url.href.includes('amenities')) {
			show('amenities');
			UIPanel.set('loaded');
		}
		if ($page.url.href.includes('overview')) {
			show('overview');
			UIPanel.set('loaded');
		}

		// Auto-advance from loading screen after 2 seconds
		setTimeout(() => {
			UIPanel.set('loaded');
			show('overview');
			goto('/overview');
		}, 2000);

	});
</script>
<div class="app">
	<slot />

	<div class={'nav-wrapper ' + ($navSlide ? 'active-drop-wrapper' : '')}>
		<nav class="z-[999]">
			{#if $UIPanel == 'instructions-nav'}
			<div class="tooltip-text relative">
				<strong> Navigation Panel</strong>
				<p>You can navigate to various options available here</p>

				<button
					on:click={() => {
						UIPanel.set('instructions-color-mode');

						!isIframe && localStorage.setItem('instructions-view-count', '2');
					}}
					class=" ghost-btn next-btn absolute bottom-[-1px] right-[-1px] border-0 !px-0 !py-0"
				>
					<svg
						width="40"
						height="40"
						viewBox="0 0 51 50"
						fill="none"
						xmlns="http://www.w3.org/2000/svg"
					>
						<path
							d="M0.00390625 12.0128C0.00390625 5.42948 5.34075 0.0926514 11.9241 0.0926514H50.0687V37.6226C50.0687 44.206 44.7318 49.5428 38.1485 49.5428H0.00390625V12.0128Z"
							fill="url(#paint0_linear_75_2649)"
						/>
						<path
							d="M33.5968 24.9588C34.2895 25.3152 34.2895 26.3054 33.5968 26.6617L20.9503 33.1676C20.1422 33.5833 19.2637 32.745 19.6413 31.9184L22.2492 26.2081C22.3646 25.9554 22.3646 25.6651 22.2492 25.4125L19.6413 19.7022C19.2637 18.8756 20.1422 18.0372 20.9503 18.4529L33.5968 24.9588Z"
							fill="white"
						/>
						<defs>
							<linearGradient
								id="paint0_linear_75_2649"
								x1="0.394398"
								y1="42.7963"
								x2="42.8438"
								y2="25.5106"
								gradientUnits="userSpaceOnUse"
							>
								<stop stop-color="#0F4C5C" />
								<stop offset="1" stop-color="#0F4C5C" />
							</linearGradient>
						</defs>
					</svg>
				</button>
			</div>
			{/if}

			<div class:hidden={$page.url.hash == '#hidenav'}>
			<svg
				on:click={() => navSlide.set(!$navSlide)}
				type="button"
				width="135"
				height="38"
				viewBox="0 0 135 38"
				fill="none"
				xmlns="http://www.w3.org/2000/svg"
				class="pointer mx-auto block cursor-pointer"
			>
				<g clip-path="url(#clip0_688_470)">
					<path
						fill-rule="evenodd"
						clip-rule="evenodd"
						d="M98.7523 13.3744C114.818 26.5965 131.663 44 152.47 44H680.754C702.293 44 719.754 61.4609 719.754 83C719.754 104.539 702.293 122 680.754 122H-492C-513.539 122 -531 104.539 -531 83C-531 61.4609 -513.539 44 -492 44H-18.4702C2.33679 44 19.1821 26.5965 35.2477 13.3744C43.8831 6.26749 54.9431 2 67 2C79.0569 2 90.117 6.26749 98.7523 13.3744Z"
						fill="url(#paint0_linear_688_470)"
					/>
					<path
						class={!$navSlide ? 'active-drop' : ''}
						d="M60.3076 24L59 22.6615L66.1266 15.3708C66.2408 15.2533 66.3766 15.1601 66.5262 15.0964C66.6757 15.0328 66.8361 15 66.9981 15C67.1602 15 67.3206 15.0328 67.4701 15.0964C67.6197 15.1601 67.7555 15.2533 67.8697 15.3708L75 22.6615L73.6924 23.9987L67 17.156L60.3076 24Z"
						fill="white"
						fill-opacity="0.7"
					/>
					<path
						class={!$navSlide ? 'active-drop' : ''}
						d="M60.3076 31L59 29.6615L66.1266 22.3708C66.2408 22.2533 66.3766 22.1601 66.5262 22.0964C66.6757 22.0328 66.8361 22 66.9981 22C67.1602 22 67.3206 22.0328 67.4701 22.0964C67.6197 22.1601 67.7555 22.2533 67.8697 22.3708L75 29.6615L73.6924 30.9987L67 24.156L60.3076 31Z"
						fill="white"
						fill-opacity="0.2"
					/>
				</g>
				<defs>
					<linearGradient
						id="paint0_linear_688_470"
						x1="-521.244"
						y1="95.6284"
						x2="-435.661"
						y2="-263.156"
						gradientUnits="userSpaceOnUse"
					>
						<stop stop-color="#0F4C5C" />
						<stop offset="1" stop-color="#0F4C5C" />
					</linearGradient>
					<clipPath id="clip0_688_470">
						<rect width="135" height="38" fill="white" />
					</clipPath>
				</defs>
			</svg>
			</div>

			<div class="btn-group" class:hidden={$page.url.hash == '#hidenav'}>
			<button
				id="overview"
				on:click={() => {
					show('overview');
					goto('/overview');
					UIPanel.set('loaded');
				}}
				class={$currentUI.overview && $UIPanel != 'Brochure'
					? 'transparent-btn active'
					: 'transparent-btn'}
			>
				<svg
					width="19"
					height="18"
					viewBox="0 0 19 18"
					fill="none"
					xmlns="http://www.w3.org/2000/svg"
				>
					<path
						d="M6.58949 15.1462C6.52087 15.2551 6.48655 15.3368 6.43165 15.398C6.26009 15.5954 6.04048 15.6363 5.81402 15.5342C5.58755 15.4253 5.45716 15.221 5.54637 14.9692C5.73853 14.3906 5.95127 13.812 6.1846 13.247C6.30813 12.9543 6.63753 12.8726 6.91204 13.0564C7.39242 13.3831 7.85908 13.7303 8.31888 14.0842C8.54534 14.2612 8.60024 14.5063 8.44927 14.7513C8.26398 15.0441 7.98947 15.0577 7.62575 14.8807C7.91398 15.541 8.19535 16.1196 8.75122 16.5144C9.01887 16.6982 9.28651 16.6778 9.54729 16.5076C9.84238 16.317 10.0483 16.0447 10.2198 15.7452C10.8169 14.7582 11.0914 13.6554 11.311 12.5458C11.3659 12.2531 11.1394 12.362 11.0296 12.3757C10.0345 12.4642 9.03945 12.471 8.05123 12.4233C6.2944 12.3416 4.57188 12.0966 2.91798 11.4771C2.26603 11.2321 1.64154 10.9257 1.11311 10.4492C0.0905803 9.53028 0.0837177 8.40711 1.10625 7.48134C1.77879 6.8755 2.60917 6.53515 3.45327 6.25606C4.13953 6.03142 4.83952 5.86124 5.55324 5.76594C5.79343 5.73191 5.86892 5.6298 5.91009 5.40517C6.08852 4.29561 6.36989 3.21327 6.864 2.1922C7.13164 1.63402 7.45419 1.11668 7.94143 0.715059C8.70318 0.0819969 9.58846 0.088804 10.3434 0.721866C10.9061 1.19836 11.2561 1.81781 11.5443 2.4781C12.162 3.91441 12.4296 5.42559 12.5668 6.964C12.6835 8.24374 12.6904 9.52348 12.5737 10.8032C12.5463 11.1163 12.6286 11.1504 12.9237 11.0891C13.8433 10.8917 14.7492 10.6603 15.5933 10.2518C15.9776 10.068 16.3344 9.85022 16.6227 9.53028C16.9727 9.14228 16.9864 8.80192 16.6501 8.40711C16.2658 7.95784 15.758 7.69236 15.1746 7.4473C15.2158 7.56302 15.2433 7.6379 15.2639 7.70597C15.3394 7.95784 15.257 8.16205 15.0305 8.28458C14.8109 8.40711 14.5639 8.4003 14.406 8.19609C14.0354 7.70597 13.6786 7.20905 13.3423 6.69171C13.157 6.40581 13.308 6.08588 13.658 5.96335C14.1727 5.77956 14.6943 5.60938 15.2158 5.4392C15.5041 5.3439 15.7442 5.42559 15.8746 5.69787C16.005 5.97696 15.9021 6.2016 15.6413 6.35816C15.607 6.37859 15.5658 6.40581 15.5109 6.44666C15.806 6.59641 16.0874 6.73936 16.3619 6.88912C16.7874 7.12056 17.1648 7.41327 17.4668 7.78766C18.0707 8.54325 18.057 9.40775 17.4256 10.1429C17.0276 10.6126 16.5129 10.9257 15.9639 11.1912C14.9345 11.6813 13.8433 11.9604 12.7247 12.1374C12.4845 12.1783 12.3884 12.2667 12.3472 12.5118C12.1482 13.7099 11.8394 14.8807 11.2629 15.9698C11.0296 16.4055 10.7551 16.8071 10.3845 17.1338C9.5816 17.835 8.64828 17.8282 7.85908 17.1134C7.37183 16.671 7.04243 16.1128 6.77478 15.5206C6.71988 15.4116 6.65812 15.2959 6.58949 15.1462ZM5.63559 8.74746C5.615 8.00549 5.66304 7.50857 5.69735 7.01165C5.70421 6.89593 5.74539 6.74617 5.52579 6.79382C4.366 7.02526 3.2268 7.29755 2.21113 7.93061C1.11311 8.61813 1.11998 9.28523 2.20427 9.97955C2.87681 10.4084 3.61797 10.6671 4.38659 10.8645C6.60322 11.4363 8.85416 11.5043 11.1257 11.341C11.4414 11.3206 11.5237 11.2048 11.5443 10.9121C11.661 9.39414 11.661 7.87615 11.4688 6.36497C11.311 5.09885 11.0502 3.85995 10.5149 2.69593C10.3159 2.26028 10.0757 1.84504 9.71199 1.52511C9.32082 1.17794 8.97769 1.17794 8.57966 1.5183C8.29143 1.77016 8.08555 2.08329 7.90712 2.41684C7.42673 3.32219 7.17968 4.29561 6.9738 5.28264C6.9189 5.5345 6.99439 5.57534 7.22086 5.55492C8.08555 5.46643 8.9571 5.46643 9.82179 5.48685C10.2267 5.49366 10.4532 5.67745 10.46 5.99058C10.4669 6.31732 10.2267 6.51473 9.80121 6.50792C8.9571 6.49431 8.10614 6.50792 7.26203 6.57599C6.7542 6.61683 6.76792 6.61003 6.72675 7.09333C6.63753 8.05314 6.65126 9.01975 6.66498 9.98636C6.67184 10.3744 6.50028 10.5922 6.1846 10.6058C5.85519 10.6194 5.64245 10.4016 5.63559 10.0136C5.62873 9.50305 5.63559 9.00613 5.63559 8.74746Z"
						fill="currentColor"
						stroke="currentColor"
						stroke-width="0.394234"
					/>
				</svg>
				Overview
			</button>
		 
			</div>
		</nav>
	</div>
</div>
