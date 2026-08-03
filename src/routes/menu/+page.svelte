<script>
	import { goto } from '$app/navigation';
	import { getContext, onMount } from 'svelte';
	import { flip } from 'svelte/animate';
	import { fade } from 'svelte/transition';

	const currentUI = getContext('currentUI');
	const UIPanel = getContext('UIPanel');

	let currentSlide = $state(1);
	let card1W = $state(420);
	let card1H = $state(231);
	let card2W = $state(420);
	let card2H = $state(231);

	import views1Video from '$lib/videos/views1.mp4';
	import vicinityVideo from '$lib/videos/vicinity.mp4';

	let canvas;
	let gl;
	let program;
	let positionBuffer;
	let texture;
	let ripples = $state([]);
	let rippleId = 0;
	let lastMouseX = 0;
	let lastMouseY = 0;
	const SPAWN_DISTANCE = 30;
	let imageLoaded = $state(false);
	let imageWidth = 0;
	let imageHeight = 0;
	let uScale = 1.0;
	let vScale = 1.0;
	let uOffset = 0.0;
	let vOffset = 0.0;
	let animationFrameId;

	const vertexShaderSource = `
		attribute vec2 a_position;
		varying vec2 v_texCoord;
		uniform vec2 u_uvScale;
		uniform vec2 u_uvOffset;
		void main() {
			gl_Position = vec4(a_position, 0.0, 1.0);
			vec2 uv = a_position * 0.5 + 0.5;
			uv.y = 1.0 - uv.y;
			v_texCoord = uv * u_uvScale + u_uvOffset;
		}
	`;

	const fragmentShaderSource = `
		precision mediump float;
		varying vec2 v_texCoord;
		uniform sampler2D u_image;
		uniform float u_aspectRatio;

		uniform vec4 u_ripples[30];
		uniform int u_rippleCount;

		void main() {
			vec2 tc = v_texCoord;
			vec2 displacement = vec2(0.0);

			for (int i = 0; i < 30; i++) {
				if (i >= u_rippleCount) break;
				
				vec4 ripple = u_ripples[i];
				vec2 ripplePos = ripple.xy;
				float progress = ripple.z;
				float intensity = ripple.w;

				vec2 diff = tc - ripplePos;
				diff.y /= u_aspectRatio;

				float dist = length(diff);
				
				float waveRadius = progress * 0.5;
				float waveWidth = 0.07;
				
				if (dist > 0.0 && dist < waveRadius + waveWidth && dist > waveRadius - waveWidth) {
					float d = dist - waveRadius;
					float x = d / waveWidth;
					float wave = sin(x * 3.14159) * (1.0 - progress);
					displacement += normalize(diff) * wave * 0.012 * intensity;
				}
			}

			displacement.y *= u_aspectRatio;
			gl_FragColor = texture2D(u_image, tc - displacement);
		}
	`;

	function createShader(gl, type, source) {
		const shader = gl.createShader(type);
		gl.shaderSource(shader, source);
		gl.compileShader(shader);
		if (!gl.getShaderParameter(shader, gl.COMPILE_STATUS)) {
			console.error('Shader compile error:', gl.getShaderInfoLog(shader));
			gl.deleteShader(shader);
			return null;
		}
		return shader;
	}

	function initWebGL() {
		if (!canvas) return;
		gl = canvas.getContext('webgl');
		if (!gl) {
			console.error('WebGL not supported');
			return;
		}

		const vs = createShader(gl, gl.VERTEX_SHADER, vertexShaderSource);
		const fs = createShader(gl, gl.FRAGMENT_SHADER, fragmentShaderSource);
		program = gl.createProgram();
		gl.attachShader(program, vs);
		gl.attachShader(program, fs);
		gl.linkProgram(program);

		if (!gl.getProgramParameter(program, gl.LINK_STATUS)) {
			console.error('Program link error:', gl.getProgramInfoLog(program));
			return;
		}

		positionBuffer = gl.createBuffer();
		gl.bindBuffer(gl.ARRAY_BUFFER, positionBuffer);
		gl.bufferData(gl.ARRAY_BUFFER, new Float32Array([
			-1.0, -1.0,
			 1.0, -1.0,
			-1.0,  1.0,
			-1.0,  1.0,
			 1.0, -1.0,
			 1.0,  1.0,
		]), gl.STATIC_DRAW);

		texture = gl.createTexture();
		gl.bindTexture(gl.TEXTURE_2D, texture);
		
		gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_WRAP_S, gl.CLAMP_TO_EDGE);
		gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_WRAP_T, gl.CLAMP_TO_EDGE);
		gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_MIN_FILTER, gl.LINEAR);
		gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_MAG_FILTER, gl.LINEAR);

		const img = new Image();
		img.src = '/building.png';
		img.onload = () => {
			gl.bindTexture(gl.TEXTURE_2D, texture);
			gl.texImage2D(gl.TEXTURE_2D, 0, gl.RGBA, gl.RGBA, gl.UNSIGNED_BYTE, img);
			imageWidth = img.naturalWidth;
			imageHeight = img.naturalHeight;
			imageLoaded = true;
			resizeCanvas();
			updateUVScale(imageWidth, imageHeight);
			requestRender();
		};
	}

	function updateUVScale(imgWidth, imgHeight) {
		if (!canvas) return;
		const canvasWidth = canvas.width;
		const canvasHeight = canvas.height;

		const imageRatio = imgWidth / imgHeight;
		const canvasRatio = canvasWidth / canvasHeight;

		if (canvasRatio > imageRatio) {
			uScale = 1.0;
			vScale = imageRatio / canvasRatio;
			uOffset = 0.0;
			vOffset = (1.0 - vScale) * 0.5;
		} else {
			uScale = canvasRatio / imageRatio;
			vScale = 1.0;
			uOffset = (1.0 - uScale) * 0.5;
			vOffset = 0.0;
		}
	}

	function resizeCanvas() {
		if (!canvas || !gl) return;
		const displayWidth = window.innerWidth;
		const displayHeight = window.innerHeight;
		if (canvas.width !== displayWidth || canvas.height !== displayHeight) {
			canvas.width = displayWidth;
			canvas.height = displayHeight;
			gl.viewport(0, 0, canvas.width, canvas.height);
		}
	}

	function requestRender() {
		if (animationFrameId) cancelAnimationFrame(animationFrameId);
		animationFrameId = requestAnimationFrame(render);
	}

	function render() {
		if (!gl || !imageLoaded) return;

		resizeCanvas();

		gl.clearColor(0.0, 0.0, 0.0, 1.0);
		gl.clear(gl.COLOR_BUFFER_BIT);

		gl.useProgram(program);

		const positionLoc = gl.getAttribLocation(program, 'a_position');
		gl.enableVertexAttribArray(positionLoc);
		gl.bindBuffer(gl.ARRAY_BUFFER, positionBuffer);
		gl.vertexAttribPointer(positionLoc, 2, gl.FLOAT, false, 0, 0);

		const uvScaleLoc = gl.getUniformLocation(program, 'u_uvScale');
		gl.uniform2f(uvScaleLoc, uScale, vScale);

		const uvOffsetLoc = gl.getUniformLocation(program, 'u_uvOffset');
		gl.uniform2f(uvOffsetLoc, uOffset, vOffset);

		const aspectLoc = gl.getUniformLocation(program, 'u_aspectRatio');
		gl.uniform1f(aspectLoc, canvas.width / canvas.height);

		const maxRipples = 30;
		const ripplesData = new Float32Array(maxRipples * 4);
		const count = Math.min(ripples.length, maxRipples);

		for (let i = 0; i < count; i++) {
			const r = ripples[i];
			const idx = i * 4;
			ripplesData[idx] = r.x / canvas.width;
			ripplesData[idx + 1] = 1.0 - (r.y / canvas.height);
			ripplesData[idx + 2] = r.progress;
			ripplesData[idx + 3] = r.intensity;
		}

		const ripplesLoc = gl.getUniformLocation(program, 'u_ripples');
		gl.uniform4fv(ripplesLoc, ripplesData);

		const countLoc = gl.getUniformLocation(program, 'u_rippleCount');
		gl.uniform1i(countLoc, count);

		gl.drawArrays(gl.TRIANGLES, 0, 6);

		if (ripples.length > 0) {
			const now = Date.now();
			ripples = ripples.map(r => {
				const elapsed = now - r.startTime;
				const duration = r.type === 'click' ? 1200 : 800;
				return {
					...r,
					progress: elapsed / duration
				};
			}).filter(r => r.progress < 1.0);

			animationFrameId = requestAnimationFrame(render);
		} else {
			animationFrameId = null;
		}
	}

	function handleResize() {
		resizeCanvas();
		if (imageWidth && imageHeight) {
			updateUVScale(imageWidth, imageHeight);
		}
		requestRender();
	}

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

		initWebGL();

		window.addEventListener('resize', handleResize);

		return () => {
			window.removeEventListener('resize', handleResize);
			if (animationFrameId) cancelAnimationFrame(animationFrameId);
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

	function spawnRipple(x, y, type) {
		const id = rippleId++;
		const intensity = type === 'click' ? 0.5 : 0.15;
		ripples = [...ripples, {
			id,
			x,
			y,
			type,
			startTime: Date.now(),
			progress: 0.0,
			intensity
		}];

		if (!animationFrameId) {
			requestRender();
		}
	}

	function handlePageClick(e) {
		if (e.target.closest('.slider-menu')) {
			return;
		}
		spawnRipple(e.clientX, e.clientY, 'click');
	}

	function handleMouseMove(e) {
		if (e.target.closest('.slider-menu')) {
			return;
		}
		const distance = Math.hypot(e.clientX - lastMouseX, e.clientY - lastMouseY);
		if (distance > SPAWN_DISTANCE) {
			lastMouseX = e.clientX;
			lastMouseY = e.clientY;
			spawnRipple(e.clientX, e.clientY, 'hover');
		}
	}
</script>

<div class="menu-container fixed inset-0 w-screen h-screen bg-black overflow-hidden select-none" on:click={handlePageClick} on:mousemove={handleMouseMove}>
	<!-- Settled Parallax Background Building -->
	<div class="fixed inset-0 pointer-events-none flex items-center justify-center" style="z-index: 1;">
		{#if !imageLoaded}
			<img class="w-full h-full object-cover scale-110 -translate-y-10" src="/building.png" alt="Building background" />
		{/if}
		<canvas
			bind:this={canvas}
			class="w-full h-full scale-110 -translate-y-10"
			class:hidden={!imageLoaded}
		></canvas>
	</div>

	<!-- Subtle progressive gradient blur columns behind side items -->
	<div 
		class="fixed left-0 top-0 w-[40vw] h-screen backdrop-blur-[6px] bg-black/5 pointer-events-none z-[5]"
		style="mask-image: linear-gradient(to right, black 60%, transparent 100%); -webkit-mask-image: linear-gradient(to right, black 60%, transparent 100%);"
	></div>
	<div 
		class="fixed right-0 top-0 w-[40vw] h-screen backdrop-blur-[6px] bg-black/5 pointer-events-none z-[5]"
		style="mask-image: linear-gradient(to left, black 60%, transparent 100%); -webkit-mask-image: linear-gradient(to left, black 60%, transparent 100%);"
	></div>

	<!-- Bottom Dark Overlay Gradient -->
	<div class="fixed bottom-0 left-0 w-full h-[20rem] bg-gradient-to-t z-10 from-black/75 via-black/35 to-transparent pointer-events-none"></div>

	<!-- Navigation UI Description Overlay -->
	<div class="menu-desc-container fixed bottom-6 left-0 max-w-[720px] z-[25] flex flex-col gap-2 pointer-events-auto text-left animate-fade-in">
		<p class="text-white/80 text-xs md:text-[16px] text-justify font-normal leading-relaxed tracking-wide normal-case pl-16" style="font-family: 'Imprima', sans-serif;">
			In the heart of South Mumbai, where heritage meets contemporary living, Raheja SOBO Residences presents a rare collection of thoughtfully crafted homes. An address defined by timeless architecture, exceptional views, and a neighbourhood that has shaped the city's finest lifestyles.
		</p>
	</div>

	<!-- Bottom Right Navigation Card & Controls -->
	<div class="slider-menu fixed bottom-12 right-6 z-[25] flex items-end gap-10 pointer-events-auto animate-fade-in">
		<div class="relative w-[420px] h-[255px]">
			<!-- Toggle/Next circular button -->
			<button
				on:click={() => currentSlide = currentSlide === 1 ? 2 : 1}
				class="toggle-slide-btn absolute -left-12 top-1/2 -translate-y-1/2 w-10 h-10 rounded-full bg-black/40 hover:bg-black/60 transition-all duration-300 flex items-center justify-center text-[#DEAD66] cursor-pointer z-30 !p-0"
				aria-label="Next slide"
			>
				<svg width="8" height="8" viewBox="0 0 10 10" fill="#DEAD66">
					<circle cx="5" cy="2" r="1.1" />
					<circle cx="2" cy="6.8" r="1.1" />
					<circle cx="8" cy="6.8" r="1.1" />
				</svg>
			</button>

			<!-- Card 2 (VICINITY) -->
			<div 
				bind:clientWidth={card2W} 
				bind:clientHeight={card2H}
				class="nav-card {currentSlide === 2 ? 'front' : 'back'} cursor-pointer"
				on:click={() => currentSlide !== 2 && (currentSlide = 2)}
			>
				<div class="glass-blur-bg"></div>
				<svg class="absolute inset-0 w-full h-full pointer-events-none" style="z-index: -1;">
					<path d="M 40,0 L {card2W - 40},0 A 40,40 0 0 1 {card2W},40 L {card2W},{card2H - 40} A 40,40 0 0 1 {card2W - 40},{card2H} L 40,{card2H} A 40,40 0 0 1 0,{card2H - 40} L 0,40 A 40,40 0 0 1 40,0 Z" 
					      fill="rgba(255, 255, 255, 0.02)" 
					      stroke="rgba(255, 255, 255, 0.12)" 
					      stroke-width="1.2" />
				</svg>
				
				<!-- Text section -->
				<div class="flex flex-col flex-1 text-left relative z-10 max-w-[48%]">
					<h2 class="text-3xl tracking-[0.1em] text-white mb-2 font-normal uppercase" style="font-family: 'The Seasons', serif;">
						VICINITY
					</h2>
					<p class="text-white/70 text-[13px] mb-5 normal-case font-light max-w-[225px]" style="font-family: 'Imprima', sans-serif; letter-spacing: 0.02em; line-height: 1.7;">
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
				
				<!-- Thumbnail Video -->
				<div class="thumbnail-cutout z-10">
					<video 
						class="w-full h-full object-cover pointer-events-none" 
						src={vicinityVideo} 
						autoplay 
						loop 
						muted 
						playsinline
					></video>
				</div>
			</div>

			<!-- Card 1 (VIEWS) -->
			<div 
				bind:clientWidth={card1W} 
				bind:clientHeight={card1H}
				class="nav-card {currentSlide === 1 ? 'front' : 'back'} cursor-pointer"
				on:click={() => currentSlide !== 1 && (currentSlide = 1)}
			>
				<div class="glass-blur-bg"></div>
				<svg class="absolute inset-0 w-full h-full pointer-events-none" style="z-index: -1;">
					<path d="M 40,0 L {card1W - 40},0 A 40,40 0 0 1 {card1W},40 L {card1W},{card1H - 40} A 40,40 0 0 1 {card1W - 40},{card1H} L 40,{card1H} A 40,40 0 0 1 0,{card1H - 40} L 0,40 A 40,40 0 0 1 40,0 Z" 
					      fill="rgba(255, 255, 255, 0.02)" 
					      stroke="rgba(255, 255, 255, 0.12)" 
					      stroke-width="1.2" />
				</svg>

				<!-- Text section -->
				<div class="flex flex-col flex-1 text-left relative z-10 max-w-[48%]">
					<h2 class="text-3xl tracking-[0.1em] text-white mb-2 font-normal uppercase" style="font-family: 'The Seasons', serif;">
						VIEWS
					</h2>
					<p class="text-white/70 text-[13px] mb-5 normal-case font-light max-w-[225px]" style="font-family: 'Imprima', sans-serif; letter-spacing: 0.02em; line-height: 1.7;">
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
				
				<!-- Thumbnail Video -->
				<div class="thumbnail-cutout z-10">
					<video 
						class="w-full h-full object-cover pointer-events-none" 
						src={views1Video} 
						autoplay 
						loop 
						muted 
						playsinline
					></video>
				</div>
			</div>
		</div>

		<!-- Page Indicators -->
		<button 
			class="custom-indicator select-none cursor-pointer border-0 bg-transparent p-0 flex items-baseline outline-none" 
			style="font-family: 'IvyPresto Text', serif;"
			on:click={() => currentSlide = currentSlide === 1 ? 2 : 1}
			aria-label="Toggle slide"
		>
			<div class="indicator-num large">
				{#key currentSlide}
					<span transition:fade={{ duration: 250 }}>
						{currentSlide === 1 ? '01' : '02'}
					</span>
				{/key}
			</div>
			
			<div class="indicator-line"></div>
			
			<div class="indicator-num small">
				{#key currentSlide}
					<span transition:fade={{ duration: 250 }}>
						{currentSlide === 1 ? '02' : '01'}
					</span>
				{/key}
			</div>
		</button>
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
		text-transform: none !important;
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
.nav-card{
    position:absolute;
    inset:0;

    padding:24px 24px;
    border-radius:28px;

    overflow:hidden;
 


    transition:.6s cubic-bezier(.19,1,.22,1);
}

.nav-card::before{
    content:"";
    position:absolute;
    inset:0;
    border-radius:inherit;

    background:
        radial-gradient(
            ellipse at top,
            rgba(255,255,255,.24),
            transparent 55%
        );

    mix-blend-mode:screen;
    pointer-events:none;
}
 
  /* stacking states */
  .nav-card.front {
    z-index: 20;
    opacity: 1 !important;
    transform: translate(0,0) scale(1) !important;
    pointer-events: auto;
  }

	.nav-card.back {
    z-index: 10;
    opacity: 0.65 !important;
    transform: translate(0, -28px) scale(0.93) !important;
    pointer-events: none;
    filter: blur(1.5px) !important;
	}

	/* Concentric Cutout Mask for Thumbnail on Right Side */
	.thumbnail-cutout {
		position: absolute;
		top: 14px;
		bottom: 14px;
		right: 14px;
		width: 48%;
		background: transparent;
		border-radius: 2.2rem;
		overflow: hidden;
		-webkit-mask-image: radial-gradient(circle at calc(100% + 14px) calc(100% + 14px), transparent 48px, black 49px);
		mask-image: radial-gradient(circle at calc(100% + 14px) calc(100% + 14px), transparent 48px, black 49px);
	}

	.thumbnail-cutout img {
		border-top-left-radius: 2.5rem;
		border-bottom-left-radius: 2.5rem;
	}

	.custom-indicator {
		display: flex;
		align-items: baseline;
		gap: 1.5rem;
		transform: translateY(10px);
		background: transparent !important;
		border: 0 !important;
		padding: 0 !important;
	}

	.indicator-num {
		display: grid;
		grid-template-areas: "stack";
		align-items: baseline;
		color: #ffffff;
		font-weight: 300;
		font-family: 'IvyPresto Text', serif;
		line-height: 0.85;
	}

	.indicator-num > span {
		grid-area: stack;
	}

	.indicator-num.large {
		font-size: 60px;
		width: 60px;
		text-align: left;
	}

	.indicator-num.small {
		font-size: 24px;
		opacity: 1;
		width: 40px;
		text-align: left; 
	}

	.indicator-line {
		width: 40px;
		height: 1.5px;
		background: rgba(255, 255, 255, 0.45);
		align-self: center;
		transform: translateY(18px);
	}

	.toggle-slide-btn {
		border: 1.8px solid #DEAD66 !important;
	}

	.slider-menu {
		transition: all 0.4s cubic-bezier(0.16, 1, 0.3, 1);
	}

	.menu-desc-container {
		transition: all 0.4s cubic-bezier(0.16, 1, 0.3, 1);
	}

	@media (max-width: 1024px) {
		.menu-desc-container {
			max-w: 50% !important;
			padding-left: 2rem !important;
		}
		.menu-desc-container p {
			font-size: 13px !important;
			line-height: 1.5 !important;
		}
		.slider-menu {
			gap: 1.5rem !important;
			right: 1.5rem !important;
			bottom: 2rem !important;
		}
	}

	@media (max-width: 768px) {
		.menu-desc-container {
			display: none !important;
		}
		.slider-menu {
			flex-direction: column !important;
			align-items: flex-end !important;
			gap: 1rem !important;
			transform: scale(0.8) !important;
			transform-origin: bottom right !important;
			right: 16px !important;
			bottom: 16px !important;
		}
	}

	@media (max-width: 480px) {
		.slider-menu {
			flex-direction: column !important;
			align-items: flex-end !important;
			gap: 0.75rem !important;
			transform: scale(0.68) !important;
			transform-origin: bottom right !important;
			right: 12px !important;
			bottom: 12px !important;
		}
	}

	.glass-blur-bg {
		position: absolute;
		inset: -60px;
		z-index: -2;
		background: url('/building.png') no-repeat center center;
		background-size: cover;
		background-attachment: fixed;
		filter: blur(45px) brightness(0.9);
		opacity: 0.95;
		pointer-events: none;
	}

</style>
