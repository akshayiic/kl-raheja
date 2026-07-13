<script>
	import bowser from 'bowser';
	import dollhouse from '$lib/images/dollhouse.png';
	import fbhkdollhouse from '$lib/images/LT_4BHK.png';
	import { onMount } from 'svelte';
	let Marzipano;
	import minimizeBtn from '$lib/images/minimize-icon.svg';
	import maximizeBtn from '$lib/images/maximize-icon.svg';
	import { fade } from 'svelte/transition';
	import startTourImg from '$lib/images/startTourImg.svg';
	import { writable } from 'svelte/store';
	import { onDestroy } from 'svelte';
	import { setContext, getContext } from 'svelte';
	const isInteriorUnitDataMinimized = writable();
	setContext('isInteriorUnitDataMinimized', isInteriorUnitDataMinimized);

	$: isInteriorUnitDataMinimized.set(true);
	let hotspotName = getContext('hotspotName');
	let currentUI = getContext('currentUI');
	$currentUI['interiors'] = true;
	const currentbhk = writable();
	currentbhk.set('3bhk');
	let unsubscribeHotSpot;

	const isInteriorUnitMinimized = writable();
	setContext('isInteriorUnitMinimized', isInteriorUnitMinimized);
	$: isInteriorUnitMinimized.set(true);

	onMount(async () => {
		// @ts-expect-error because my dead marzipano doesn't have type bindings like i even cared in the first place
		Marzipano = await import('marzipano');
		let allScenes = {};
		let counter = 0;
		// Create viewer.
		var viewer = new Marzipano.Viewer(document.getElementById('pano'), {
			controls: {
				mouseViewMode: 'drag' // drag|qtvr
			}
		});
		window.dragControlMethod = bowser.mobile
			? viewer.controls().method('touchView').instance
			: viewer.controls().method('mouseViewDrag').instance;
		console.log('Marzipano initialized on main app', window?.dragControlMethod);
		const createHotspot = (
			scene,
			position,
			nextScene,
			tooltipText = 'Area Label',
			hotspotNameUpdate
		) => {
			// Create wrapper element to hold icon and tooltip.
			var wrapper = document.createElement('div');

			wrapper.classList.add('info-hotspot');
			wrapper.id = 'hotspot-wrapper-' + tooltipText.replaceAll(' ', '') + counter;

			// hotspots
			var imgHotspot = document.createElement('div');
			imgHotspot.innerText = tooltipText;
			imgHotspot.id = 'hotspot-' + tooltipText.replaceAll(' ', '') + counter;
			counter++;
			imgHotspot.classList.add('hotspot');

			// imgHotspot.classList.add('animate-bounce');

			wrapper.appendChild(imgHotspot);

			imgHotspot.addEventListener('click', function () {
				nextScene.switchTo();
				console.log('next scene data', nextScene);
				window.view = nextScene._view;
				// Create text element.
				$hotspotName = hotspotNameUpdate;

				console.log('hotspot name changed', hotspotNameUpdate);
			});

			scene.hotspotContainer().createHotspot(wrapper, position, {});
		};

		const create360scene = (name, size, initialView) => {
			var urlPrefix = 'https://framer-assets.vestate.iiclab.com/evara/marzi';
			let source = Marzipano.ImageUrlSource.fromString(
				urlPrefix + '/' + name + '/{z}/{f}/{y}/{x}.jpg',
				{ cubeMapPreviewUrl: urlPrefix + '/' + name + '/preview.jpg' }
			);
			var geometry = new Marzipano.CubeGeometry(size);

			// let source = new Marzipano.ImageUrlSource.fromString(path);
			var limiter = Marzipano.RectilinearView.limit.traditional(3840, (130 * Math.PI) / 180);
			// var initialView = {
			// 	yaw: 180,
			// 	pitch: 180,
			// 	fov: 180
			// };
			var view = new Marzipano.RectilinearView(initialView, limiter);

			let scene = viewer.createScene({
				source: source,
				geometry: geometry,
				view: view,
				pinFirstLevel: true
			});
			allScenes[name] = {
				source,
				view,
				scene
			};
		};

		// 4bhk scenes
		//
		create360scene(
			'fbhkbalcony2',
			[
				{ tileSize: 256, size: 256, fallbackOnly: true },
				{ tileSize: 512, size: 512 },
				{ tileSize: 512, size: 1024 },
				{ tileSize: 512, size: 2048 }
			],
			{ yaw: -1.1531113684246765, pitch: -0.0623011349777336, fov: 130 * Math.PI }
		);
		create360scene(
			'fbhkdinningarea',
			[
				{ tileSize: 256, size: 256, fallbackOnly: true },
				{ tileSize: 512, size: 512 },
				{ tileSize: 512, size: 1024 },
				{ tileSize: 512, size: 2048 }
			],
			{ yaw: 2.911788926922746, pitch: 0.0034208689761392463, fov: 130 * Math.PI }
		);
		create360scene(
			'fbhkguestbedroom2',
			[
				{ tileSize: 256, size: 256, fallbackOnly: true },
				{ tileSize: 512, size: 512 },
				{ tileSize: 512, size: 1024 },
				{ tileSize: 512, size: 2048 }
			],
			{ yaw: -2.2242583684375745, pitch: -0.02395061132081544, fov: 130 * Math.PI }
		);
		create360scene(
			'fbhkkitchen',
			[
				{ tileSize: 256, size: 256, fallbackOnly: true },
				{ tileSize: 512, size: 512 },
				{ tileSize: 512, size: 1024 },
				{ tileSize: 512, size: 2048 }
			],
			{ yaw: -2.294471093521814, pitch: 0.0311394170251873, fov: 130 * Math.PI }
		);
		create360scene(
			'fbhklivingroom',
			[
				{ tileSize: 256, size: 256, fallbackOnly: true },
				{ tileSize: 512, size: 512 },
				{ tileSize: 512, size: 1024 },
				{ tileSize: 512, size: 2048 }
			],
			{ yaw: -2.05523516558487, pitch: 0.15026707799832373, fov: 130 * Math.PI }
		);
		create360scene(
			'fbhkmasterbalcony2',
			[
				{ tileSize: 256, size: 256, fallbackOnly: true },
				{ tileSize: 512, size: 512 },
				{ tileSize: 512, size: 1024 },
				{ tileSize: 512, size: 2048 }
			],
			{ yaw: 1.8490140611666348, pitch: 0.31682277139707793, fov: 130 * Math.PI }
		);
		create360scene(
			'fbhkmasterwashroom',
			[
				{ tileSize: 256, size: 256, fallbackOnly: true },
				{ tileSize: 512, size: 512 },
				{ tileSize: 512, size: 1024 },
				{ tileSize: 512, size: 2048 }
			],
			{ yaw: 0.8849977168898189, pitch: 0.14219913500572012, fov: 130 * Math.PI }
		);
		create360scene(
			'fbhkpowderroom',
			[
				{ tileSize: 256, size: 256, fallbackOnly: true },
				{ tileSize: 512, size: 512 },
				{ tileSize: 512, size: 1024 },
				{ tileSize: 512, size: 2048 }
			],
			{ yaw: 2.572325035608551, pitch: -0.023322290535134016, fov: 130 * Math.PI }
		);
		create360scene(
			'fbhkwashroom2',
			[
				{ tileSize: 256, size: 256, fallbackOnly: true },
				{ tileSize: 512, size: 512 },
				{ tileSize: 512, size: 1024 },
				{ tileSize: 512, size: 2048 }
			],
			{ yaw: 0.4720322921451512, pitch: -0.12455897650754189, fov: 130 * Math.PI }
		);
		create360scene(
			'fbhkbedroom31',
			[
				{ tileSize: 256, size: 256, fallbackOnly: true },
				{ tileSize: 512, size: 512 },
				{ tileSize: 512, size: 1024 },
				{ tileSize: 512, size: 2048 }
			],
			{ yaw: 2.722449437330055, pitch: 0.0695670053558608, fov: 130 * Math.PI }
		);
		create360scene(
			'fbhkfoyerspace',
			[
				{ tileSize: 256, size: 256, fallbackOnly: true },
				{ tileSize: 512, size: 512 },
				{ tileSize: 512, size: 1024 },
				{ tileSize: 512, size: 2048 }
			],
			{ yaw: 0.44041502175851477, pitch: 0.13151127178827693, fov: 130 * Math.PI }
		);
		create360scene(
			'fbhkkidsroom1',
			[
				{ tileSize: 256, size: 256, fallbackOnly: true },
				{ tileSize: 512, size: 512 },
				{ tileSize: 512, size: 1024 },
				{ tileSize: 512, size: 2048 }
			],
			{ yaw: -2.0489591727415615, pitch: 0.040393853595137585, fov: 130 * Math.PI }
		);
		create360scene(
			'fbhklivingarea',
			[
				{ tileSize: 256, size: 256, fallbackOnly: true },
				{ tileSize: 512, size: 512 },
				{ tileSize: 512, size: 1024 },
				{ tileSize: 512, size: 2048 }
			],
			{ yaw: -1.9717289347999802, pitch: 0.08644827095960927, fov: 130 * Math.PI }
		);
		create360scene(
			'fbhklivingroom2',
			[
				{ tileSize: 256, size: 256, fallbackOnly: true },
				{ tileSize: 512, size: 512 },
				{ tileSize: 512, size: 1024 },
				{ tileSize: 512, size: 2048 }
			],
			{ yaw: 2.887844669857916, pitch: 0.07807095602646541, fov: 130 * Math.PI }
		);
		create360scene(
			'fbhkmasterbedroom',
			[
				{ tileSize: 256, size: 256, fallbackOnly: true },
				{ tileSize: 512, size: 512 },
				{ tileSize: 512, size: 1024 },
				{ tileSize: 512, size: 2048 }
			],
			{ yaw: -2.9048578537104373, pitch: 0.05521974618450898, fov: 130 * Math.PI }
		);
		create360scene(
			'fbhkmasterwashroom2',
			[
				{ tileSize: 256, size: 256, fallbackOnly: true },
				{ tileSize: 512, size: 512 },
				{ tileSize: 512, size: 1024 },
				{ tileSize: 512, size: 2048 }
			],
			{ yaw: 0.7472995830600304, pitch: 0.0005058521607725908, fov: 130 * Math.PI }
		);
		create360scene(
			'fbhkutilityspace',
			[
				{ tileSize: 256, size: 256, fallbackOnly: true },
				{ tileSize: 512, size: 512 },
				{ tileSize: 512, size: 1024 },
				{ tileSize: 512, size: 2048 }
			],
			{ yaw: 2.1521807183210164, pitch: -0.048229246325307784, fov: 130 * Math.PI }
		);
		create360scene(
			'fbhkwashroom4',
			[
				{ tileSize: 256, size: 256, fallbackOnly: true },
				{ tileSize: 512, size: 512 },
				{ tileSize: 512, size: 1024 },
				{ tileSize: 512, size: 2048 }
			],
			{ yaw: -2.6954958297461875, pitch: -0.03718053506091934, fov: 130 * Math.PI }
		);
		create360scene(
			'fbhkbedroom32',
			[
				{ tileSize: 256, size: 256, fallbackOnly: true },
				{ tileSize: 512, size: 512 },
				{ tileSize: 512, size: 1024 },
				{ tileSize: 512, size: 2048 }
			],
			{ yaw: 1.23476223834267, pitch: -0.03531945247125989, fov: 130 * Math.PI }
		);
		create360scene(
			'fbhkguestbedroom1',
			[
				{ tileSize: 256, size: 256, fallbackOnly: true },
				{ tileSize: 512, size: 512 },
				{ tileSize: 512, size: 1024 },
				{ tileSize: 512, size: 2048 }
			],
			{ yaw: -0.4245254693686178, pitch: -0.028275099263478154, fov: 130 * Math.PI }
		);
		create360scene(
			'fbhkkidsroom2',
			[
				{ tileSize: 256, size: 256, fallbackOnly: true },
				{ tileSize: 512, size: 512 },
				{ tileSize: 512, size: 1024 },
				{ tileSize: 512, size: 2048 }
			],
			{ yaw: -0.41744688719077594, pitch: 0.0016008912331599845, fov: 130 * Math.PI }
		);
		create360scene(
			'fbhklivingbalcony',
			[
				{ tileSize: 256, size: 256, fallbackOnly: true },
				{ tileSize: 512, size: 512 },
				{ tileSize: 512, size: 1024 },
				{ tileSize: 512, size: 2048 }
			],
			{ yaw: 1.955470215428183, pitch: 0.04482153402541833, fov: 130 * Math.PI }
		);
		create360scene(
			'fbhkmasterbalcony',
			[
				{ tileSize: 256, size: 256, fallbackOnly: true },
				{ tileSize: 512, size: 512 },
				{ tileSize: 512, size: 1024 },
				{ tileSize: 512, size: 2048 }
			],
			{ yaw: 2.0665356742182013, pitch: -0.025188033752534977, fov: 130 * Math.PI }
		);
		create360scene(
			'fbhkmasterbedroom2',
			[
				{ tileSize: 256, size: 256, fallbackOnly: true },
				{ tileSize: 512, size: 512 },
				{ tileSize: 512, size: 1024 },
				{ tileSize: 512, size: 2048 }
			],
			{ yaw: -0.2760340546609825, pitch: 0.19113813092069343, fov: 130 * Math.PI }
		);
		create360scene(
			'fbhkpoojaroom',
			[
				{ tileSize: 256, size: 256, fallbackOnly: true },
				{ tileSize: 512, size: 512 },
				{ tileSize: 512, size: 1024 },
				{ tileSize: 512, size: 2048 }
			],
			{ yaw: 0.4237084866902414, pitch: 0.012669350615826858, fov: 130 * Math.PI }
		);
		create360scene(
			'fbhkwalkinwardrobe',
			[
				{ tileSize: 256, size: 256, fallbackOnly: true },
				{ tileSize: 512, size: 512 },
				{ tileSize: 512, size: 1024 },
				{ tileSize: 512, size: 2048 }
			],
			{ yaw: 0.41725027559050787, pitch: 0.09949971612287278, fov: 130 * Math.PI }
		);
		//
		//
		// 3bhk scences
		create360scene(
			'living',
			[
				{
					tileSize: 256,
					size: 256,
					fallbackOnly: true
				},
				{
					tileSize: 512,
					size: 512
				},
				{
					tileSize: 512,
					size: 1024
				},
				{
					tileSize: 512,
					size: 2048
				}
			],
			{
				yaw: 1.972406964656809,
				pitch: 0.0646221359482606,
				fov: 130 * Math.PI
			}
		);

		create360scene(
			'foyerArea',
			[
				{
					tileSize: 256,
					size: 256,
					fallbackOnly: true
				},
				{
					tileSize: 512,
					size: 512
				},
				{
					tileSize: 512,
					size: 1024
				},
				{
					tileSize: 512,
					size: 2048
				}
			],
			{
				yaw: 2.302628705370253,
				pitch: 0.0482215426531063,
				fov: 130 * Math.PI
			}
		);
		create360scene(
			'dining',
			[
				{
					tileSize: 256,
					size: 256,
					fallbackOnly: true
				},
				{
					tileSize: 512,
					size: 512
				},
				{
					tileSize: 512,
					size: 1024
				},
				{
					tileSize: 512,
					size: 2048
				}
			],
			{ yaw: -2.277794607104955, pitch: 0.17095672570439646, fov: 130 * Math.PI }
		);

		create360scene(
			'commonWashroom',
			[
				{
					tileSize: 256,
					size: 256,
					fallbackOnly: true
				},
				{
					tileSize: 512,
					size: 512
				},
				{
					tileSize: 512,
					size: 1024
				},
				{
					tileSize: 512,
					size: 2048
				}
			],
			{
				yaw: -2.8572026943567277,
				pitch: 0.6534246538675852,
				fov: 130 * Math.PI
			}
		);

		create360scene(
			'washroom1',
			[
				{
					tileSize: 256,
					size: 256,
					fallbackOnly: true
				},
				{
					tileSize: 512,
					size: 512
				},
				{
					tileSize: 512,
					size: 1024
				},
				{
					tileSize: 512,
					size: 2048
				}
			],
			{
				yaw: -2.7405092976289183,
				pitch: 0.10073207107334525,
				fov: 130 * Math.PI
			}
		);
		create360scene(
			'washroom2',
			[
				{
					tileSize: 256,
					size: 256,
					fallbackOnly: true
				},
				{
					tileSize: 512,
					size: 512
				},
				{
					tileSize: 512,
					size: 1024
				},
				{
					tileSize: 512,
					size: 2048
				}
			],
			{
				yaw: 0.3462430714504965,
				pitch: 0.013844599443933348,
				fov: 130 * Math.PI
			}
		);

		create360scene(
			'bedroom3',
			[
				{
					tileSize: 256,
					size: 256,
					fallbackOnly: true
				},
				{
					tileSize: 512,
					size: 512
				},
				{
					tileSize: 512,
					size: 1024
				},
				{
					tileSize: 512,
					size: 2048
				}
			],
			{
				yaw: 2.28495655864819,
				pitch: -0.0055514687729445455,
				fov: 130 * Math.PI
			}
		);
		create360scene(
			'bedroom2',
			[
				{
					tileSize: 256,
					size: 256,
					fallbackOnly: true
				},
				{
					tileSize: 512,
					size: 512
				},
				{
					tileSize: 512,
					size: 1024
				},
				{
					tileSize: 512,
					size: 2048
				}
			],
			{
				yaw: 1.1885206759386886,
				pitch: 0.00867989691363924,
				fov: 130 * Math.PI
			}
		);
		create360scene(
			'mbedroom',
			[
				{
					tileSize: 256,
					size: 256,
					fallbackOnly: true
				},
				{
					tileSize: 512,
					size: 512
				},
				{
					tileSize: 512,
					size: 1024
				},
				{
					tileSize: 512,
					size: 2048
				}
			],
			{
				yaw: 1.6255259739106034,
				pitch: 0.05478945245531541,
				fov: 130 * Math.PI
			}
		);

		create360scene(
			'kitchen',
			[
				{
					tileSize: 256,
					size: 256,
					fallbackOnly: true
				},
				{
					tileSize: 512,
					size: 512
				},
				{
					tileSize: 512,
					size: 1024
				},
				{
					tileSize: 512,
					size: 2048
				}
			],
			{
				yaw: 2.2401751581216702,
				pitch: 0.017831622525051216,
				fov: 130 * Math.PI
			}
		);

		create360scene(
			'balcony',
			[
				{
					tileSize: 256,
					size: 256,
					fallbackOnly: true
				},
				{
					tileSize: 512,
					size: 512
				},
				{
					tileSize: 512,
					size: 1024
				},
				{
					tileSize: 512,
					size: 2048
				},
				{
					tileSize: 512,
					size: 4096
				}
			],
			{
				yaw: -2.5074957370646267,
				pitch: 0.010606841317125415,
				fov: 130 * Math.PI
			}
		);

		var pano = document.getElementById('pano');
		// hotspots
		//

		createHotspot(
			allScenes.fbhkfoyerspace.scene,
			{ yaw: 1.3792312114352967, pitch: 0.08540337286488864 },
			allScenes.fbhkpoojaroom.scene,
			'Pooja Room',
			'fbhkpoojaroom'
		);

		createHotspot(
			allScenes.fbhkfoyerspace.scene,
			{ yaw: 2.3213208182595224, pitch: 0.0655471282704081 },
			allScenes.fbhkdinningarea.scene,

			'Dining Area',
			'fbhkdinningarea'
		);

		createHotspot(
			allScenes.fbhkfoyerspace.scene,
			{ yaw: 2.9984054713619646, pitch: 0.37118111578164736 },
			allScenes.fbhklivingarea.scene,

			'Living Area',
			'fbhklivingarea'
		);

		createHotspot(
			allScenes.fbhkfoyerspace.scene,
			{ yaw: 1.9476449225061456, pitch: 0.04200386084268004 },
			allScenes.fbhkbedroom31.scene,

			'Bedroom 3',
			'fbhkbedroom31'
		);

		createHotspot(
			allScenes.fbhkfoyerspace.scene,
			{ yaw: 1.8083548416321573, pitch: 0.14378769414025783 },
			allScenes.fbhkguestbedroom1.scene,

			'Guest Bedroom',
			'fbhkguestbedroom1'
		);

		createHotspot(
			allScenes.fbhkfoyerspace.scene,
			{ yaw: 1.6981174933020213, pitch: 0.25319977954470296 },
			allScenes.fbhkkitchen.scene,

			'Kitchen',
			'fbhkkitchen'
		);

		createHotspot(
			allScenes.fbhkfoyerspace.scene,
			{ yaw: -0.6437918817352077, pitch: 0.27252771699903455 },
			allScenes.fbhkpowderroom.scene,

			'Powder Room',
			'fbhkpowderroom'
		);

		createHotspot(
			allScenes.fbhkfoyerspace.scene,
			{ yaw: -1.1144116915381694, pitch: 0.060655566962994456 },
			allScenes.fbhkmasterbedroom.scene,

			'Master Bedroom',
			'fbhkmasterbedroom'
		);

		createHotspot(
			allScenes.fbhkfoyerspace.scene,
			{ yaw: 2.9702496360605055, pitch: 0.09874116566491864 },
			allScenes.fbhklivingbalcony.scene,

			'Living Balcony',
			'fbhklivingbalcony'
		);

		createHotspot(
			allScenes.fbhklivingroom.scene,
			{ yaw: -2.3096370601064677, pitch: 0.3162290007280788 },
			allScenes.fbhkdinningarea.scene,

			'Dining Area',
			'fbhkdinningarea'
		);

		createHotspot(
			allScenes.fbhklivingroom.scene,
			{ yaw: -2.1607965108830314, pitch: 0.13500153885270194 },
			allScenes.fbhklivingbalcony.scene,

			'Living Balcony',
			'fbhklivingbalcony'
		);

		createHotspot(
			allScenes.fbhklivingroom.scene,
			{ yaw: -1.617617838714093, pitch: 0.16779747681300705 },
			allScenes.fbhklivingarea.scene,

			'Living Area',
			'fbhklivingarea'
		);

		createHotspot(
			allScenes.fbhklivingroom.scene,
			{ yaw: -1.0252758590989934, pitch: 0.3049978202935417 },
			allScenes.fbhkpoojaroom.scene,

			'Pooja Room',
			'fbhkpoojaroom'
		);

		createHotspot(
			allScenes.fbhklivingroom.scene,
			{ yaw: -0.7630997766413437, pitch: 0.4479163570244804 },
			allScenes.fbhkkitchen.scene,

			'Kitchen',
			'fbhkkitchen'
		);

		createHotspot(
			allScenes.fbhklivingroom.scene,
			{ yaw: -1.1816888518887652, pitch: 0.03277180623377518 },
			allScenes.fbhkmasterbedroom.scene,

			'Master Bedroom',
			'fbhkmasterbedroom'
		);

		createHotspot(
			allScenes.fbhklivingroom.scene,
			{ yaw: -1.1816888518887652, pitch: 0.03277180623377518 },
			allScenes.fbhkmasterbedroom.scene,

			'Master Bedroom',
			'fbhkmasterbedroom'
		);

		createHotspot(
			allScenes.fbhklivingroom.scene,
			{ yaw: -1.1096803972257057, pitch: 0.13713995535271906 },
			allScenes.fbhkpowderroom.scene,

			'Powder Room',
			'fbhkpowderroom'
		);

		createHotspot(
			allScenes.fbhklivingroom.scene,
			{ yaw: -1.2228323352143669, pitch: 0.13116202975789903 },
			allScenes.fbhkkidsroom1.scene,

			'Kids Room',
			'fbhkkidsroom1'
		);

		createHotspot(
			allScenes.fbhklivingroom.scene,
			{ yaw: 2.415500575877523, pitch: 0.4480943257001311 },
			allScenes.fbhkbedroom31.scene,
			'Bedroom 3',
			'fbhkbedroom31'
		);

		createHotspot(
			allScenes.fbhklivingroom.scene,
			{ yaw: 1.3514502697958601, pitch: 0.4234777475207636 },
			allScenes.fbhkguestbedroom1.scene,
			'Guest Bedroom',
			'fbhkguestbedroom1'
		);

		createHotspot(
			allScenes.fbhklivingroom2.scene,
			{ yaw: 2.442231959821094, pitch: 0.11971328271092219 },
			allScenes.fbhkdinningarea.scene,
			'Dining Area',
			'fbhkdinningarea'
		);

		createHotspot(
			allScenes.fbhklivingroom2.scene,
			{ yaw: 3.1111031204110606, pitch: 0.11643820521387127 },
			allScenes.fbhklivingbalcony.scene,
			'Living Balcony',
			'fbhklivingbalcony'
		);

		createHotspot(
			allScenes.fbhklivingroom2.scene,
			{ yaw: 2.9576917252472965, pitch: 0.298931761080361 },
			allScenes.fbhklivingarea.scene,
			'Living Area',
			'fbhklivingarea'
		);

		createHotspot(
			allScenes.fbhklivingroom2.scene,
			{ yaw: 2.0820430464883497, pitch: 0.10392227507231766 },
			allScenes.fbhkbedroom31.scene,
			'Bedroom 3',
			'fbhkbedroom31'
		);

		createHotspot(
			allScenes.fbhklivingroom2.scene,
			{ yaw: 1.942510372356729, pitch: 0.21006302883859007 },
			allScenes.fbhkkitchen.scene,
			'Kitchen',
			'fbhkkitchen'
		);

		createHotspot(
			allScenes.fbhklivingroom2.scene,
			{ yaw: 1.877088449604658, pitch: 0.5193365578109486 },
			allScenes.fbhkpoojaroom.scene,
			'Pooja Room',
			'fbhkpoojaroom'
		);

		createHotspot(
			allScenes.fbhklivingroom2.scene,

			{ yaw: 0.34397882798210766, pitch: 0.7828576859283931 },
			allScenes.fbhkfoyerspace.scene,
			'Foyer Space',
			'fbhkfoyerspace'
		);

		createHotspot(
			allScenes.fbhklivingroom2.scene,

			{ yaw: -1.4818520260420645, pitch: 0.1955043021904963 },
			allScenes.fbhkkidsroom1.scene,
			'Kids Room',
			'fbhkkidsroom1'
		);

		createHotspot(
			allScenes.fbhklivingroom2.scene,

			{ yaw: -1.2692714567454004, pitch: 0.091139974724598 },
			allScenes.fbhkmasterbedroom.scene,
			'Master Bedroom',
			'fbhkmasterbedroom'
		);

		createHotspot(
			allScenes.fbhklivingroom2.scene,

			{ yaw: -1.1402669004513637, pitch: 0.2119433941938773 },
			allScenes.fbhkpowderroom.scene,
			'Powder Room',
			'fbhkpowderroom'
		);

		createHotspot(
			allScenes.fbhkwashroom4.scene,

			{ yaw: 1.288769442008423, pitch: 0.03019596577361483 },
			allScenes.fbhkguestbedroom1.scene,
			'Guest bedroom',
			'fbhkguestbedroom1'
		);

		createHotspot(
			allScenes.fbhkwashroom2.scene,

			{ yaw: -1.3440789217472897, pitch: 0.03530722455130686 },
			allScenes.fbhkkidsroom1.scene,
			'Kids Room',
			'fbhkkidsroom1'
		);

		createHotspot(
			allScenes.fbhkwalkinwardrobe.scene,

			{ yaw: 1.1644772150360794, pitch: 0.011428899057809971 },
			allScenes.fbhkmasterwashroom.scene,
			'Master Washroom',
			'fbhkmasterwashroom'
		);

		createHotspot(
			allScenes.fbhkwalkinwardrobe.scene,

			{ yaw: 1.9559876583708, pitch: 0.057658203619528337 },
			allScenes.fbhklivingarea.scene,
			'Living Area',
			'fbhklivingarea'
		);

		createHotspot(
			allScenes.fbhkwalkinwardrobe.scene,

			{ yaw: 2.053665210750287, pitch: 0.18990127950090852 },
			allScenes.fbhkkidsroom1.scene,
			'Kids Room',
			'fbhkkidsroom1'
		);

		createHotspot(
			allScenes.fbhkwalkinwardrobe.scene,

			{ yaw: 1.840923004414635, pitch: 0.09333657133684525 },
			allScenes.fbhkpowderroom.scene,
			'Powder Room',
			'fbhkpowderroom'
		);

		createHotspot(
			allScenes.fbhkwalkinwardrobe.scene,

			{ yaw: -2.8511889481743804, pitch: 0.06540095093367171 },
			allScenes.fbhkmasterbalcony.scene,
			'Master Balcony',
			'fbhkmasterbalcony'
		);

		createHotspot(
			allScenes.fbhkutilityspace.scene,

			{ yaw: 3.021509330685503, pitch: 0.05678317454840709 },
			allScenes.fbhkkitchen.scene,
			'Kitchen',
			'fbhkkitchen'
		);

		createHotspot(
			allScenes.fbhkutilityspace.scene,

			{ yaw: -2.8392115314543425, pitch: 0.04856977453486522 },
			allScenes.fbhkdinningarea.scene,
			'Dining Area',
			'fbhkdinningarea'
		);

		createHotspot(
			allScenes.fbhkpowderroom.scene,

			{ yaw: -2.143535308038322, pitch: 0.10314135553928239 },
			allScenes.fbhklivingroom.scene,
			'Living Room',
			'fbhklivingroom'
		);

		createHotspot(
			allScenes.fbhkpoojaroom.scene,

			{ yaw: -0.27063532271028734, pitch: 0.06294209384048344 },
			allScenes.fbhkfoyerspace.scene,
			'Foyer Space',
			'fbhkfoyerspace'
		);

		createHotspot(
			allScenes.fbhkpoojaroom.scene,

			{ yaw: -0.6608775775269216, pitch: 0.07368174911782432 },
			allScenes.fbhkpowderroom.scene,
			'Powder Room',
			'fbhkpowderroom'
		);

		createHotspot(
			allScenes.fbhkpoojaroom.scene,

			{ yaw: 1.2623170213416852, pitch: 0.08610341130166788 },
			allScenes.fbhkkitchen.scene,
			'Kitchen',
			'fbhkkitchen'
		);

		createHotspot(
			allScenes.fbhkpoojaroom.scene,
			{ yaw: 1.6218612501040646, pitch: -0.0002617694938997772 },
			allScenes.fbhkguestbedroom1.scene,
			'Guest Bedroom',
			'fbhkguestbedroom1'
		);

		createHotspot(
			allScenes.fbhkpoojaroom.scene,
			{ yaw: 1.7852896455555491, pitch: 0.03531625950036599 },
			allScenes.fbhkbedroom31.scene,
			'Bedroom 3',
			'fbhkbedroom31'
		);

		createHotspot(
			allScenes.fbhkpoojaroom.scene,
			{ yaw: 2.2377156148044843, pitch: 0.17728297021614026 },
			allScenes.fbhkdinningarea.scene,
			'Dining Area',
			'fbhkdinningarea'
		);

		createHotspot(
			allScenes.fbhkpoojaroom.scene,
			{ yaw: -2.943421958116655, pitch: 0.1976333901700098 },
			allScenes.fbhklivingbalcony.scene,
			'Living Balcony',
			'fbhklivingbalcony'
		);

		createHotspot(
			allScenes.fbhkpoojaroom.scene,
			{ yaw: -2.3143509614845748, pitch: 0.31969307352525433 },
			allScenes.fbhklivingarea.scene,
			'Living Area',
			'fbhklivingarea'
		);

		createHotspot(
			allScenes.fbhkmasterwashroom.scene,
			{ yaw: -1.479325466045644, pitch: 0.09743832218102355 },
			allScenes.fbhkmasterbedroom.scene,
			'Master Bedroom',
			'fbhkmasterbedroom'
		);

		createHotspot(
			allScenes.fbhkmasterbedroom2.scene,
			{ yaw: 0.10559545293170025, pitch: 0.058374565247923726 },
			allScenes.fbhkwalkinwardrobe.scene,
			'Walk In Wardrobe',
			'fbhkwalkinwardrobe'
		);

		createHotspot(
			allScenes.fbhkmasterbedroom2.scene,
			{ yaw: -1.7292640974052347, pitch: 0.1926238722560445 },
			allScenes.fbhkmasterbalcony.scene,
			'Master Balcony',
			'fbhkmasterbalcony'
		);

		createHotspot(
			allScenes.fbhkmasterbedroom2.scene,
			{ yaw: 0.5176358490598041, pitch: 0.2498931538905218 },
			allScenes.fbhklivingarea.scene,
			'Living Area',
			'fbhklivingarea'
		);

		createHotspot(
			allScenes.fbhkmasterbedroom.scene,
			{ yaw: -2.776368553868716, pitch: 0.029501095092202334 },
			allScenes.fbhkmasterbalcony.scene,
			'Master Balcony',
			'fbhkmasterbalcony'
		);

		createHotspot(
			allScenes.fbhkmasterbedroom.scene,
			{ yaw: -3.018345866771572, pitch: 0.1016619290001195 },
			allScenes.fbhkmasterbedroom2.scene,
			'Master Bedroom 2',
			'fbhkmasterbedroom2'
		);

		createHotspot(
			allScenes.fbhkmasterbedroom.scene,
			{ yaw: 2.3253068323110737, pitch: 0.22318899644390555 },
			allScenes.fbhklivingarea.scene,
			'Living Area',
			'fbhklivingarea'
		);

		createHotspot(
			allScenes.fbhkmasterbedroom.scene,
			{ yaw: 1.6533720256010325, pitch: 0.08509937143707269 },
			allScenes.fbhkmasterwashroom.scene,
			'Master Washroom',
			'fbhkmasterwashroom'
		);

		createHotspot(
			allScenes.fbhkmasterbedroom.scene,
			{ yaw: 0.5227110490828792, pitch: 0.12124439624332695 },
			allScenes.fbhkwalkinwardrobe.scene,
			'Walk In Wardrobe',
			'fbhkwalkinwardrobe'
		);

		createHotspot(
			allScenes.fbhkmasterbalcony2.scene,
			{ yaw: 0.04111478761761056, pitch: 0.13849057675277976 },
			allScenes.fbhkbedroom31.scene,
			'Bedroom 3',
			'fbhkbedroom31'
		);

		createHotspot(
			allScenes.fbhkmasterbalcony2.scene,
			{ yaw: 0.18405495992124266, pitch: -0.16243831177403223 },
			allScenes.fbhkbedroom32.scene,
			'Bedroom 3 view 2',
			'fbhkbedroom32'
		);

		createHotspot(
			allScenes.fbhkmasterbalcony2.scene,
			{ yaw: 0.30386959670510016, pitch: 0.04128409108432152 },
			allScenes.fbhkguestbedroom1.scene,
			'Guest Bedroom',
			'fbhkguestbedroom1'
		);

		createHotspot(
			allScenes.fbhkmasterbalcony2.scene,
			{ yaw: 0.2615712672160022, pitch: 0.1783118824276304 },
			allScenes.fbhklivingarea.scene,
			'Living Area',
			'fbhklivingarea'
		);
		createHotspot(
			allScenes.fbhkmasterbalcony.scene,
			{ yaw: 1.2397487711818496, pitch: 0.13301052578128036 },
			allScenes.fbhkmasterbedroom.scene,
			'Master Bedroom',
			'fbhkmasterbedroom'
		);

		createHotspot(
			allScenes.fbhkmasterbalcony.scene,
			{ yaw: 0.5986106549571808, pitch: 0.016484196928063355 },
			allScenes.fbhkwalkinwardrobe.scene,
			'Walk In Wardrobe',
			'fbhkwalkinwardrobe'
		);

		createHotspot(
			allScenes.fbhkmasterbalcony.scene,
			{ yaw: 0.8264914633918075, pitch: 0.05676188096266088 },
			allScenes.fbhklivingarea.scene,
			'Living Area',
			'fbhklivingarea'
		);

		createHotspot(
			allScenes.fbhklivingbalcony.scene,
			{ yaw: 1.4925081779424136, pitch: 0.0968824526785994 },
			allScenes.fbhkdinningarea.scene,
			'Dining Area',
			'fbhkdinningarea'
		);

		createHotspot(
			allScenes.fbhklivingbalcony.scene,
			{ yaw: 0.9670464623099448, pitch: 0.03089073257567776 },
			allScenes.fbhkkitchen.scene,
			'Kitchen',
			'fbhkkitchen'
		);

		createHotspot(
			allScenes.fbhklivingbalcony.scene,
			{ yaw: 1.221360824310917, pitch: 0.0110880197247063 },
			allScenes.fbhkguestbedroom1.scene,
			'Guest Bedroom',
			'fbhkguestbedroom1'
		);

		createHotspot(
			allScenes.fbhklivingbalcony.scene,
			{ yaw: 0.96546422224732, pitch: 0.2323948539485663 },
			allScenes.fbhklivingarea.scene,
			'Living Area',
			'fbhklivingarea'
		);

		createHotspot(
			allScenes.fbhklivingbalcony.scene,
			{ yaw: 0.96546422224732, pitch: 0.2323948539485663 },
			allScenes.fbhklivingarea.scene,
			'Living Area',
			'fbhklivingarea'
		);

		createHotspot(
			allScenes.fbhklivingarea.scene,
			{ yaw: -2.5678057156480847, pitch: 0.05030204696781482 },
			allScenes.fbhklivingbalcony.scene,
			'Living Balcony',
			'fbhklivingbalcony'
		);

		createHotspot(
			allScenes.fbhklivingarea.scene,
			{ yaw: -3.1007243822811805, pitch: 0.1136646961760519 },
			allScenes.fbhkdinningarea.scene,
			'Dining Area',
			'fbhkdinningarea'
		);
		createHotspot(
			allScenes.fbhklivingarea.scene,
			{ yaw: 0.0665686329467956, pitch: 0.016883322581287885 },
			allScenes.fbhkkitchen.scene,
			'Kitchen',
			'fbhkkitchen'
		);

		createHotspot(
			allScenes.fbhklivingarea.scene,
			{ yaw: -0.9256646599981941, pitch: -0.000406321899202311 },
			allScenes.fbhkpoojaroom.scene,
			'Pooja Room',
			'fbhkpoojaroom'
		);

		createHotspot(
			allScenes.fbhklivingarea.scene,
			{ yaw: -1.0292518854575903, pitch: 0.022762042712908936 },
			allScenes.fbhkfoyerspace.scene,
			'Foyer Space',
			'fbhkfoyerspace'
		);

		createHotspot(
			allScenes.fbhklivingarea.scene,
			{ yaw: -1.192105475408603, pitch: -0.005671234200590902 },
			allScenes.fbhkmasterbedroom.scene,
			'Master Bedroom',
			'fbhkmasterbedroom'
		);

		createHotspot(
			allScenes.fbhklivingarea.scene,
			{ yaw: -1.1160859634351006, pitch: 0.05416726650051551 },
			allScenes.fbhkpowderroom.scene,
			'Powder Room',
			'fbhkpowderroom'
		);

		createHotspot(
			allScenes.fbhklivingarea.scene,
			{ yaw: -1.2631322103458427, pitch: 0.043179002846184034 },
			allScenes.fbhkkidsroom1.scene,
			'Kids Room',
			'fbhkkidsroom1'
		);
		createHotspot(
			allScenes.fbhklivingarea.scene,
			{ yaw: 0.3977817529068659, pitch: 0.030474389369018695 },
			allScenes.fbhkutilityspace.scene,
			'Utility Space',
			'fbhkutilityspace'
		);

		createHotspot(
			allScenes.fbhklivingarea.scene,
			{ yaw: 1.7640847486310909, pitch: -0.0800712210634309 },
			allScenes.fbhkguestbedroom1.scene,
			'Guest Bedroom',
			'fbhkguestbedroom1'
		);

		createHotspot(
			allScenes.fbhklivingarea.scene,
			{ yaw: 2.169109047307556, pitch: -0.040956063254078856 },
			allScenes.fbhkbedroom31.scene,
			'Bedroom 3',
			'fbhkbedroom31'
		);

		createHotspot(
			allScenes.fbhkkitchen.scene,
			{ yaw: 0.7249829200591886, pitch: 0.08144154672510595 },
			allScenes.fbhkutilityspace.scene,
			'Utility Space',
			'fbhkutilityspace'
		);
		createHotspot(
			allScenes.fbhkkitchen.scene,
			{ yaw: -2.8934576462528767, pitch: 0.04497865715860172 },
			allScenes.fbhkdinningarea.scene,
			'Dining Area',
			'fbhkdinningarea'
		);
		createHotspot(
			allScenes.fbhkkitchen.scene,
			{ yaw: -2.7111074939065993, pitch: -0.0014676390048364851 },
			allScenes.fbhklivingbalcony.scene,
			'Living Balcony',
			'fbhklivingbalcony'
		);
		createHotspot(
			allScenes.fbhkkitchen.scene,
			{ yaw: -2.606087718188002, pitch: 0.17380904234578942 },
			allScenes.fbhklivingarea.scene,
			'Living Area',
			'fbhklivingarea'
		);

		// Kids Room 2 hotspots
		createHotspot(
			allScenes.fbhkkidsroom2.scene,
			{ yaw: -2.5854635134126056, pitch: 0.3333991954800837 },
			allScenes.fbhkbalcony2.scene,
			'Balcony 2',
			'fbhkbalcony2'
		);
		createHotspot(
			allScenes.fbhkkidsroom2.scene,
			{ yaw: -0.2615947869430393, pitch: -0.06268670297649237 },
			allScenes.fbhkkidsroom1.scene,
			'Kids Room 1',
			'fbhkkidsroom1'
		);
		createHotspot(
			allScenes.fbhkkidsroom2.scene,
			{ yaw: 0.4552005676496318, pitch: 0.1522898437351934 },
			allScenes.fbhklivingarea.scene,
			'Living Area',
			'fbhklivingarea'
		);
		createHotspot(
			allScenes.fbhkkidsroom2.scene,
			{ yaw: 0.8792997873464365, pitch: 0.11870874713413926 },
			allScenes.fbhkwashroom2.scene,
			'Washroom 2',
			'fbhkwashroom2'
		);

		// Kids Room 1 hotspots
		createHotspot(
			allScenes.fbhkkidsroom1.scene,
			{ yaw: 1.9219010416396447, pitch: 0.0062626413703839034 },
			allScenes.fbhkwashroom2.scene,
			'Washroom 2',
			'fbhkwashroom2'
		);
		createHotspot(
			allScenes.fbhkkidsroom1.scene,
			{ yaw: -2.602187542842861, pitch: 0.108962105022675 },
			allScenes.fbhkbalcony2.scene,
			'Balcony 2',
			'fbhkbalcony2'
		);
		createHotspot(
			allScenes.fbhkkidsroom1.scene,
			{ yaw: -3.0530488798582276, pitch: -0.10798271272850002 },
			allScenes.fbhkkidsroom2.scene,
			'Kids Room 2',
			'fbhkkidsroom2'
		);
		createHotspot(
			allScenes.fbhkkidsroom1.scene,
			{ yaw: 0.4156013185215848, pitch: 0.5761927255045105 },
			allScenes.fbhklivingarea.scene,
			'Living Area',
			'fbhklivingarea'
		);

		// Guest Bedroom 1 hotspots
		createHotspot(
			allScenes.fbhkguestbedroom1.scene,
			{ yaw: 0.35221412484025727, pitch: -0.1873869157921515 },
			allScenes.fbhkguestbedroom2.scene,
			'Guest Bedroom 2',
			'fbhkguestbedroom2'
		);
		createHotspot(
			allScenes.fbhkguestbedroom1.scene,
			{ yaw: -1.3272581949642266, pitch: 0.00996879087861302 },
			allScenes.fbhkwashroom4.scene,
			'Washroom 4',
			'fbhkwashroom4'
		);
		createHotspot(
			allScenes.fbhkguestbedroom1.scene,
			{ yaw: -1.5249888647722898, pitch: 0.37420586305786685 },
			allScenes.fbhklivingarea.scene,
			'Living Area',
			'fbhklivingarea'
		);

		createHotspot(
			allScenes.fbhkguestbedroom2.scene,
			{ yaw: -2.9659330340203063, pitch: -0.09488195779695907 },
			allScenes.fbhkguestbedroom1.scene,
			'Guest Bedroom 1',
			'fbhkguestbedroom1'
		);

		createHotspot(
			allScenes.fbhkguestbedroom2.scene,
			{ yaw: -1.9859416119567967, pitch: -0.044332149734804815 },
			allScenes.fbhkwashroom4.scene,
			'Washroom 4',
			'fbhkwashroom4'
		);

		createHotspot(
			allScenes.fbhkguestbedroom2.scene,
			{ yaw: -2.2909105969288017, pitch: 0.1736555603787835 },
			allScenes.fbhklivingarea.scene,
			'Living Area',
			'fbhklivingarea'
		);

		createHotspot(
			allScenes.fbhkguestbedroom2.scene,
			{ yaw: -2.226707177398648, pitch: 0.021669470873412422 },
			allScenes.fbhkdinningarea.scene,
			'Dining Area',
			'fbhkdinningarea'
		);

		createHotspot(
			allScenes.fbhkdinningarea.scene,
			{ yaw: 1.659647253980749, pitch: 0.003959963158749247 },
			allScenes.fbhkguestbedroom1.scene,
			'Guest Bedroom 1',
			'fbhkguestbedroom1'
		);

		createHotspot(
			allScenes.fbhkdinningarea.scene,
			{ yaw: 1.9940628216278968, pitch: 0.04572399537739713 },
			allScenes.fbhkbedroom31.scene,
			'Bedroom 3',
			'fbhkbedroom31'
		);

		createHotspot(
			allScenes.fbhkdinningarea.scene,
			{ yaw: -2.5349183818474046, pitch: 0.10772333853834226 },
			allScenes.fbhklivingbalcony.scene,
			'Living Balcony',
			'fbhklivingbalcony'
		);

		createHotspot(
			allScenes.fbhkdinningarea.scene,
			{ yaw: -1.877871957497721, pitch: 0.1689580588026498 },
			allScenes.fbhklivingarea.scene,
			'Living Area',
			'fbhklivingarea'
		);

		createHotspot(
			allScenes.fbhkdinningarea.scene,
			{ yaw: -0.7320904740021987, pitch: 0.16639688640819017 },
			allScenes.fbhkpoojaroom.scene,
			'Pooja Room',
			'fbhkpoojaroom'
		);

		createHotspot(
			allScenes.fbhkdinningarea.scene,
			{ yaw: -0.9290643098845948, pitch: 0.08537238144256065 },
			allScenes.fbhkfoyerspace.scene,
			'Foyer Space',
			'fbhkfoyerspace'
		);

		createHotspot(
			allScenes.fbhkdinningarea.scene,
			{ yaw: -1.158037306357695, pitch: 0.01364233290882666 },
			allScenes.fbhkmasterbedroom.scene,
			'Master Bedroom',
			'fbhkmasterbedroom'
		);

		createHotspot(
			allScenes.fbhkdinningarea.scene,
			{ yaw: -1.024211426060699, pitch: 0.11756325319274907 },
			allScenes.fbhkpowderroom.scene,
			'Powder Room',
			'fbhkpowderroom'
		);

		createHotspot(
			allScenes.fbhkdinningarea.scene,
			{ yaw: -1.2080039928456685, pitch: 0.1504990410468494 },
			allScenes.fbhkkidsroom1.scene,
			'Kids Room',
			'fbhkkidsroom1'
		);
		createHotspot(
			allScenes['fbhkbedroom32'].scene,
			{ yaw: 2.5152588892117276, pitch: 0.27356676864176777 },
			allScenes['fbhkmasterbalcony2'].scene,
			'Master Balcony 2',
			'fbhkmasterbalcony2'
		);
		createHotspot(
			allScenes['fbhkbedroom32'].scene,
			{ yaw: 0.2972884394302717, pitch: -0.21069459698624904 },
			allScenes['fbhkbedroom31'].scene,
			'Bedroom 3',
			'fbhkbedroom31'
		);
		createHotspot(
			allScenes['fbhkbedroom32'].scene,
			{ yaw: 0.6562406639211975, pitch: 0.06832270842212829 },
			allScenes['fbhkmasterwashroom'].scene,
			'Master Washroom',
			'fbhkmasterwashroom'
		);
		createHotspot(
			allScenes['fbhkbedroom32'].scene,
			{ yaw: 0.38323711893380796, pitch: 0.22306870417159708 },
			allScenes['fbhklivingarea'].scene,
			'Living Area',
			'fbhklivingarea'
		);
		createHotspot(
			allScenes['fbhkbedroom32'].scene,
			{ yaw: 0.3965796406123161, pitch: -0.010577149235315275 },
			allScenes['fbhkguestbedroom1'].scene,
			'Guest Bedroom 1',
			'fbhkguestbedroom1'
		);
		createHotspot(
			allScenes['fbhkbedroom31'].scene,
			{ yaw: -2.968064063917181, pitch: 0.10984713158167381 },
			allScenes['fbhkmasterbalcony2'].scene,
			'Master Balcony 2',
			'fbhkmasterbalcony2'
		);
		createHotspot(
			allScenes['fbhkbedroom31'].scene,
			{ yaw: -2.6701552519358493, pitch: -0.07587494469647815 },
			allScenes['fbhkbedroom32'].scene,
			'Bedroom 3 (2)',
			'fbhkbedroom32'
		);
		createHotspot(
			allScenes['fbhkbedroom31'].scene,
			{ yaw: 1.259431829724818, pitch: 0.12044296768462459 },
			allScenes['fbhkmasterwashroom2'].scene,
			'Master Washroom 2',
			'fbhkmasterwashroom2'
		);
		createHotspot(
			allScenes['fbhkmasterwashroom2'].scene,
			{ yaw: -2.3760879611555232, pitch: 0.31381110914290034 },
			allScenes['fbhkbedroom31'].scene,
			'Bedroom 3',
			'fbhkbedroom31'
		);
		createHotspot(
			allScenes['fbhkbedroom31'].scene,
			{ yaw: 0.4132896732432947, pitch: 0.020479366016418865 },
			allScenes['fbhkguestbedroom1'].scene,
			'Guest Bedroom 1',
			'fbhkguestbedroom1'
		);
		createHotspot(
			allScenes['fbhkbedroom31'].scene,
			{ yaw: 0.2392418959760576, pitch: 0.5123391698406632 },
			allScenes['fbhklivingarea'].scene,
			'Living Area',
			'fbhklivingarea'
		);
		createHotspot(
			allScenes.fbhkbalcony2.scene,
			{ yaw: 0.7618836022999105, pitch: 0.2621506266542877 },
			allScenes.fbhkkidsroom2.scene,
			'Kids Room',
			'fbhkkidsroom2'
		);

		createHotspot(
			allScenes.fbhkbalcony2.scene,
			{ yaw: -0.024825552488248803, pitch: -0.06797397190199206 },
			allScenes.fbhkkidsroom1.scene,
			'Kids Room',
			'fbhkkidsroom1'
		);

		createHotspot(
			allScenes.fbhkbalcony2.scene,
			{ yaw: 0.4497579226293098, pitch: 0.0647830485313392 },
			allScenes.fbhklivingarea.scene,
			'Living Area',
			'fbhklivingarea'
		);

		createHotspot(
			allScenes.fbhkbalcony2.scene,
			{ yaw: 0.6474156404076101, pitch: 0.06352504372262402 },
			allScenes.fbhkwashroom2.scene,
			'Washroom 2',
			'fbhkwashroom2'
		);

		// 3bhk hotspot
		createHotspot(
			allScenes.foyerArea.scene,
			{ yaw: 2.0172303324338863, pitch: 0.3198635275540074 },
			allScenes.living.scene,

			'Living Room',
			'living'
		);

		createHotspot(
			allScenes.foyerArea.scene,
			{ yaw: -2.295237765315271, pitch: 0.34048522755086097 },
			allScenes.living.scene,

			'Living Room',
			'living'
		);

		// swp

		createHotspot(
			allScenes.foyerArea.scene,
			{ yaw: 1.6555113586406511, pitch: 0.19817642638853705 },
			allScenes.balcony.scene,

			'Balcony',
			'balcony'
		);

		createHotspot(
			allScenes.foyerArea.scene,
			{ yaw: 3.11912318722856, pitch: 0.44172178800500106 },
			allScenes.dining.scene,

			'Dining Area',
			'dining'
		);

		createHotspot(
			allScenes.foyerArea.scene,
			{ yaw: 2.7366319858179056, pitch: 0.14100657560845775 },
			allScenes.kitchen.scene,

			'Kitchen',
			'kitchen'
		);

		createHotspot(
			allScenes.foyerArea.scene,
			{ yaw: 2.9447312033083417, pitch: 0.09980241782209731 },
			allScenes.mbedroom.scene,

			'Master Bedroom',
			'mbedroom'
		);

		createHotspot(
			allScenes.foyerArea.scene,
			{ yaw: 2.8530024152797644, pitch: -0.02039484039673667 },
			allScenes.bedroom3.scene,

			'Bedroom 3',
			'bedroom3'
		);

		createHotspot(
			allScenes.foyerArea.scene,
			{ yaw: 0.9819096297322343, pitch: 0.4707923099282336 },
			allScenes.bedroom2.scene,

			'Bedroom 2',
			'bedroom2'
		);

		createHotspot(
			allScenes.balcony.scene,
			{ yaw: -1.8593677284163945, pitch: 0.3184982515447139 },
			allScenes.living.scene,

			'Living Room',
			'living'
		);

		createHotspot(
			allScenes.balcony.scene,
			{ yaw: -1.5446144397457182, pitch: 0.060660317267961616 },
			allScenes.dining.scene,

			'Dining Area',
			'dining'
		);

		createHotspot(
			allScenes.bedroom2.scene,
			{ yaw: -2.666854519153757, pitch: 0.48733352843063216 },
			allScenes.living.scene,

			'Living Room',
			'living'
		);

		createHotspot(
			allScenes.bedroom2.scene,
			{ yaw: -2.2366743560113562, pitch: 0.07083240972858462 },
			allScenes.dining.scene,

			'Dining Area',
			'dining'
		);

		createHotspot(
			allScenes.bedroom2.scene,
			{ yaw: -2.7128111367462076, pitch: -0.004020235650660453 },
			allScenes.mbedroom.scene,

			'Master Bedroom',
			'mbedroom'
		);

		createHotspot(
			allScenes.bedroom2.scene,
			{ yaw: -2.5759414882795966, pitch: -0.000025095641367656185 },
			allScenes.commonWashroom.scene,

			'Washroom 3',
			'commonWashroom'
		);

		createHotspot(
			allScenes.bedroom3.scene,
			{ yaw: 1.0588949810441264, pitch: 0.2064220041515732 },
			allScenes.living.scene,

			'Living Room',
			'living'
		);

		createHotspot(
			allScenes.commonWashroom.scene,
			{ yaw: 1.3536961488833885, pitch: 0.7180739357786692 },
			allScenes.living.scene,

			'Living Room',
			'living'
		);

		createHotspot(
			allScenes.dining.scene,
			{ yaw: -2.69873678142117, pitch: 0.2232289654355597 },
			allScenes.commonWashroom.scene,

			'Washroom 3',
			'commonWashroom'
		);
		createHotspot(
			allScenes.dining.scene,
			{ yaw: 1.9490804228184109, pitch: 0.1609643989297549 },
			allScenes.balcony.scene,

			'Balcony',
			'balcony'
		);

		createHotspot(
			allScenes.dining.scene,
			{ yaw: 0.6318826222340252, pitch: 0.3048438707775283 },
			allScenes.bedroom2.scene,

			'Bedroom 2',
			'bedroom2'
		);

		createHotspot(
			allScenes.dining.scene,
			{ yaw: -2.8029410338001366, pitch: 0.12332470798204298 },
			allScenes.mbedroom.scene,

			'Master Bedroom',
			'mbedroom'
		);

		createHotspot(
			allScenes.dining.scene,
			{ yaw: 2.3893889946325952, pitch: 0.2779232862405614 },
			allScenes.living.scene,

			'Living Room',
			'living'
		);

		createHotspot(
			allScenes.dining.scene,
			{ yaw: -2.991315779189364, pitch: 0.13060770123535903 },
			allScenes.kitchen.scene,

			'Kitchen',
			'kitchen'
		);
		createHotspot(
			allScenes.dining.scene,
			{ yaw: -2.912228587952608, pitch: 0.00593917288315815 },
			allScenes.bedroom3.scene,

			'Bedroom 3',
			'bedroom3'
		);

		createHotspot(
			allScenes.kitchen.scene,
			{
				yaw: -2.8572026943567277,
				pitch: 0.6534246538675852
			},
			allScenes.commonWashroom.scene,

			'Washroom 3',
			'commonWashroom'
		);

		createHotspot(
			allScenes.washroom2.scene,
			{
				yaw: 1.4930029343719324,
				pitch: -0.5470672860657615
			},
			allScenes.living.scene,

			'Living Room',
			'living'
		);

		createHotspot(
			allScenes.washroom1.scene,
			{
				yaw: 1.7925911076224619,
				pitch: 0.018311098419408367
			},
			allScenes.living.scene,

			'Living Room',
			'living'
		);
		// update
		createHotspot(
			allScenes.kitchen.scene,
			{ yaw: -0.664208346789227, pitch: 0.22917597825418312 },
			allScenes.dining.scene,

			'Dining Area',
			'dining'
		);

		createHotspot(
			allScenes.kitchen.scene,
			{ yaw: -0.712555725113134, pitch: 0.5764875684416886 },
			allScenes.living.scene,

			'Living room',
			'living'
		);

		createHotspot(
			allScenes.living.scene,
			{ yaw: 1.6638176406195253, pitch: 0.10347170493858115 },
			allScenes.balcony.scene,

			'Balcony',
			'balcony'
		);

		createHotspot(
			allScenes.living.scene,
			{ yaw: 0.542711314320691, pitch: 0.17243136823337935 },
			allScenes.bedroom2.scene,

			'Bedroom 2',
			'bedroom2'
		);

		createHotspot(
			allScenes.living.scene,
			{ yaw: -0.20149058451188928, pitch: 0.06328301190653107 },
			allScenes.foyerArea.scene,

			'Foyer Area',
			'foyerArea'
		);

		createHotspot(
			allScenes.living.scene,
			{ yaw: -3.0792706961951897, pitch: 0.08039656090181424 },
			allScenes.kitchen.scene,

			'Kitchen',
			'kitchen'
		);

		createHotspot(
			allScenes.living.scene,
			{ yaw: -2.8008969845013816, pitch: 0.08170185679392361 },
			allScenes.mbedroom.scene,

			'Master Bedroom',
			'mbedroom'
		);

		createHotspot(
			allScenes.mbedroom.scene,
			{ yaw: -0.9350487049820604, pitch: 0.2087219278678969 },
			allScenes.washroom1.scene,

			'Washroom 1',
			'washroom1'
		);

		createHotspot(
			allScenes.mbedroom.scene,
			{ yaw: -0.9350487049820604, pitch: 0.2087219278678969 },
			allScenes.washroom1.scene,

			'Washroom 1',
			'washroom1'
		);
		createHotspot(
			allScenes.mbedroom.scene,
			{ yaw: 0.2436874534165554, pitch: 0.06161603421005779 },
			allScenes.dining.scene,

			'Dining Area',
			'dining'
		);

		createHotspot(
			allScenes.mbedroom.scene,
			{ yaw: 0.17021959159466782, pitch: 0.1166028868656781 },
			allScenes.commonWashroom.scene,

			'Washroom 3',
			'commonWashroom'
		);
		createHotspot(
			allScenes.mbedroom.scene,
			{ yaw: 0.2954546501123634, pitch: 0.16367328482425947 },
			allScenes.living.scene,

			'Living Area',
			'living'
		);

		createHotspot(
			allScenes.living.scene,
			{ yaw: -2.6349657029918063, pitch: 0.16204061858999808 },
			allScenes.commonWashroom.scene,

			'Washroom 3',
			'commonWashroom'
		);
		createHotspot(
			allScenes.living.scene,
			{ yaw: -1.9357892125268368, pitch: 0.2989147997773358 },
			allScenes.dining.scene,

			'Dining Area',
			'dining'
		);

		// -- generated hotspots
		//
		//

		pano.addEventListener('click', function (e) {
			var imgHotspot = document.createElement('img');
			imgHotspot.src = '';
			imgHotspot.classList.add('hotspot');
			imgHotspot.addEventListener('click', function () {
				alert('CLICK!');
			});
			// // http://www.marzipano.net/reference/Viewer.html#view dapat dari view di index.js
			var view = viewer.view();
			var loc = view.screenToCoordinates({ x: e.clientX, y: e.clientY });
			var position = { yaw: loc._yaw, pitch: loc._pitch };
			// http://www.marzipano.net/reference/Viewer.html#scene dapat dari scene
			console.log(view.screenToCoordinates({ x: e.clientX, y: e.clientY }));
			console.log(view._yaw + ' - ' + view._pitch);
			var sceness = viewer.scene();
			sceness.hotspotContainer().createHotspot(imgHotspot, position);
		});
		hotspotName.set('fbhkfoyerspace');
		unsubscribeHotSpot = hotspotName.subscribe((changedHotspot) => {
			// console.log('$currentUI.interiors', $currentUI.interiors, changedHotspot);
			if ($currentUI.interiors) {
				allScenes[changedHotspot].scene.switchTo();
				window.view = allScenes[changedHotspot].view;
				// window.dragControlMethod.addEventListener('parameterDynamics', resetArrowValues);
			}
		});

		setTimeout(() => {
			window?.marzipano.next();
		}, 4000);
	});

	function resetArrowValues() {
		document.querySelectorAll('.info-hotspot').forEach((e) => {
			console.log('LISTENER');
			setTimeout(() => {
				e.style.transform = resetNegativeTranslations(
					e.style.transform,
					e.querySelector('.hotspot')
				);
			}, 500);
		});
	}
	onDestroy(() => {
		if (unsubscribeHotSpot) unsubscribeHotSpot();
	});
	function resetNegativeTranslations(inputString, element) {
		// Regular expression to match translate values
		const translateRegex = /translate[XYZ]?\((-?\d+(\.\d+)?px)\)/g;

		// Replace negative or large translate values
		const transformedString = inputString.replace(translateRegex, (match, value) => {
			const numericValue = parseFloat(value);
			if (numericValue < 0) {
				setTimeout(() => {
					if (match.charAt(9) == 'X') {
						element.classList.add('left-arrow');
					} else {
						element.classList.add('top-arrow');
					}
				}, 200);

				return `translate${match.charAt(9)}(100px)`;
			} else if (match.includes('translateX') && numericValue > window.innerWidth) {
				setTimeout(() => {
					element.classList.add('right-arrow');
				}, 200);

				return `translateX(${window.innerWidth - 250}px)`;
			} else if (match.includes('translateY') && numericValue > window.innerHeight) {
				setTimeout(() => {
					element.classList.add('down-arrow');
				}, 200);

				return `translateY(${window.innerHeight - 200}px)`;
			}
			element.classList.remove('left-arrow');
			element.classList.remove('right-arrow');
			element.classList.remove('down-arrow');
			element.classList.remove('top-arrow');
			return match;
		});

		return transformedString;
	}
</script>

{#if $currentbhk == '3bhk' && !['foyerArea', 'living', 'bedroom1', 'bedroom2', 'bedroom3', 'bedroom4', 'mbedroom', 'commonWashroom', 'washroom1', 'washroom2', 'balcony', 'washroom4', 'kitchen', 'dining'].includes($hotspotName)}
	<div class="dollhouse-wrapper z-[9]" transition:fade>
		<img src={dollhouse} id="dollhouse" class="pointer relative z-[9000]" alt="dollhouse" />
		<button
			id="start-tour"
			data-vretail-product-id="3bhk-start-tour" data-vretail-name="start tour"
			class="primary-btn mx-auto mt-[-1rem]"
			on:click={() => ($hotspotName = 'foyerArea')}
		>
			<img id="startTourImg" src={startTourImg} alt="start tour" />
			Start Tour
		</button>
	</div>
{/if}

<div class="bhk-selector fixed right-5 top-8 z-[999]">
	<label class="relative inline-flex cursor-pointer items-center">
		<input
			on:change={() => ($currentbhk == '4bhk' ? ($currentbhk = '3bhk') : ($currentbhk = '4bhk'))}
			type="checkbox"
			class="peer sr-only"
		/>
		<div
			class="peer h-10 w-[8.5rem] rounded-full bg-gray-200 after:absolute
		after:left-[4px] after:top-[4px]
		after:h-8 after:w-[4rem] after:rounded-full after:bg-[#0F4C5C]
		after:transition-all after:content-[''] peer-checked:bg-[#f5f5f5] peer-checked:after:translate-x-16 peer-checked:after:border-white
		peer-focus:outline-none"
		></div>
		<span
			class={'absolute left-4 text-sm font-medium   ' +
				($currentbhk == '3bhk' ? 'text-white' : 'text-black')}
			>3 BHK
		</span>
		<span
			class={'absolute right-4 text-sm font-medium text-black ' +
				($currentbhk == '4bhk' ? 'text-white' : 'text-black')}>4 BHK</span
		>
	</label>
</div>
<div class="left-panel-wrapper">
	{#if $currentbhk == '3bhk'}
		<div class="left-panel p-2">
			<div class="left-panel--header flex justify-between gap-[3rem]">
				<div class="left-title flex items-center font-bold">
					<svg
						width="25"
						height="25"
						viewBox="0 0 32 31"
						class="mr-2"
						fill="none"
						xmlns="http://www.w3.org/2000/svg"
					>
						<path
							d="M15.7834 30.1853C11.9139 30.1853 8.05047 30.1677 4.18106 30.1971C3.40362 30.203 2.89323 29.6084 2.89323 28.9079C2.89917 25.4642 2.89917 22.0205 2.89917 18.5768C2.89917 16.7225 2.89917 14.8682 2.89917 13.0139C2.89917 12.9079 2.95258 12.7666 2.85763 12.7078C2.76267 12.6489 2.67365 12.7784 2.58463 12.8314C2.00897 13.2023 1.4333 13.579 0.851705 13.9499C0.786424 13.9911 0.697404 14.0499 0.643991 14.0323C0.543102 13.997 0.584645 13.8792 0.584645 13.7968C0.584645 12.59 0.584645 11.3833 0.57871 10.1765C0.57871 9.98225 0.649926 9.8704 0.804228 9.77033C2.9051 8.38696 5.00598 7.00359 7.11279 5.62611C8.3294 4.82552 9.54601 4.03082 10.7626 3.23612C12.3412 2.20007 13.9258 1.17578 15.4985 0.133843C15.7181 -0.0133237 15.8902 -0.00743706 16.1098 0.13973C17.5103 1.06982 18.9169 1.98815 20.3234 2.90647C22.8931 4.59594 25.4687 6.28542 28.0385 7.96901C28.9524 8.56945 29.8604 9.16989 30.7803 9.75856C30.9583 9.8704 31.0295 9.99991 31.0236 10.2118C31.0117 11.4009 31.0177 12.5842 31.0177 13.7733C31.0177 13.8616 31.0533 13.9793 30.9761 14.0205C30.8812 14.0794 30.7981 13.9734 30.7209 13.9204C30.1334 13.5437 29.5518 13.1611 28.9643 12.7843C28.899 12.7431 28.8218 12.6371 28.7388 12.696C28.6972 12.7254 28.7031 12.8432 28.7031 12.9197C28.7031 15.6158 28.7031 18.3119 28.7031 21.0139C28.7031 23.6511 28.6913 26.2883 28.7091 28.9315C28.715 29.6202 28.1097 30.1853 27.4391 30.1853C23.5518 30.1735 19.6646 30.1853 15.7834 30.1853ZM15.8071 27.7894C19.1661 27.7894 22.5311 27.7894 25.8901 27.7894C26.2818 27.7894 26.2818 27.7894 26.2818 27.4009C26.2818 22.0382 26.2818 16.6695 26.2877 11.3068C26.2877 11.0889 26.2046 10.9771 26.0325 10.8653C24.6201 9.95282 23.2136 9.0345 21.813 8.11029C19.8842 6.84465 17.9495 5.57313 16.0267 4.30161C15.8605 4.18976 15.7418 4.19565 15.5816 4.3075C14.8219 4.81964 14.0564 5.31412 13.2908 5.81448C10.7092 7.50396 8.12762 9.19932 5.54604 10.8829C5.37393 10.9948 5.32052 11.1184 5.32645 11.3185C5.33239 16.6636 5.33239 22.0087 5.33239 27.3538C5.33239 27.7953 5.33239 27.7953 5.78936 27.7953C9.11871 27.7894 12.4599 27.7894 15.8071 27.7894Z"
							fill="#0F4C5C"
						/>
						<path
							d="M20.4084 26.7112C20.3193 26.7112 20.2303 26.7112 20.1413 26.7112C19.4173 26.7112 19.2036 26.5229 19.1739 25.8223C19.1621 25.5751 19.0137 25.5928 18.8535 25.5928C17.536 25.5928 16.2244 25.5928 14.9069 25.5928C14.1829 25.5928 13.4648 25.5986 12.7407 25.5869C12.5034 25.581 12.4321 25.6634 12.4381 25.893C12.444 26.4404 12.1592 26.7112 11.6013 26.7112C11.2927 26.7112 10.99 26.7171 10.6814 26.7112C10.2601 26.7053 9.93958 26.3992 9.90991 25.9813C9.90991 25.9401 9.90397 25.9048 9.90991 25.8635C9.99299 25.2337 9.8565 24.7098 9.37579 24.2271C8.97223 23.8209 8.82386 23.2499 8.78232 22.6671C8.77045 22.4728 8.74078 22.2786 8.72891 22.0843C8.72297 21.9783 8.65176 21.9371 8.56867 21.89C7.80903 21.4956 7.49449 20.8952 7.60132 20.024C7.67847 19.3705 8.30161 18.7171 9.0019 18.5523C9.95145 18.3345 10.9485 18.9879 11.1443 19.9357C11.2155 20.2712 11.1681 20.6185 11.2571 20.9541C11.4648 21.737 11.8031 22.0078 12.628 22.0078C14.7407 22.0078 16.8594 22.0078 18.9722 22.0078C19.9158 22.0078 20.3015 21.6605 20.4084 20.5714C20.4558 20.1005 20.4855 19.6413 20.7941 19.2469C21.2333 18.6818 21.803 18.4463 22.5033 18.5405C23.1502 18.6288 23.6131 19.0115 23.8742 19.5825C24.2303 20.3654 24.0344 21.4603 23.0374 21.8842C22.8831 21.9489 22.895 22.0725 22.8831 22.202C22.8416 22.9202 22.711 23.6207 22.2422 24.2035C22.1294 24.3389 22.0404 24.5155 21.898 24.6038C21.5953 24.7922 21.5597 25.0571 21.5775 25.3632C21.5894 25.6104 21.5834 25.8518 21.5834 26.099C21.5775 26.4758 21.3579 26.6936 20.9721 26.7053C20.7822 26.7171 20.5923 26.7171 20.4084 26.7112Z"
							fill="#0F4C5C"
						/>
						<path
							d="M15.9302 14.0137C17.1171 14.0137 18.304 14.0255 19.491 14.0137C21.2417 13.9901 22.8797 15.4795 22.945 17.216C22.945 17.2631 22.9509 17.3161 22.9687 17.3632C23.0221 17.5339 22.9806 17.5987 22.7847 17.5692C22.0666 17.4515 21.3841 17.5869 20.7729 17.9695C19.9954 18.4522 19.5919 19.1704 19.5028 20.0711C19.491 20.177 19.4791 20.283 19.4613 20.389C19.3604 20.9482 19.3604 20.9482 18.8026 20.9482C16.7729 20.9482 14.7492 20.9482 12.7195 20.9482C12.3159 20.9482 12.3219 20.9482 12.2803 20.5538C12.2151 19.9298 12.1557 19.3058 11.7581 18.776C11.0815 17.8812 10.2151 17.4044 9.05781 17.5633C8.89164 17.5869 8.84416 17.5339 8.85009 17.3809C8.90351 16.4743 9.24772 15.6855 9.88273 15.0438C10.5059 14.414 11.2714 14.0372 12.1676 14.0196C13.4198 13.996 14.6779 14.0137 15.9302 14.0137Z"
							fill="#0F4C5C"
						/>
					</svg>

					Interior Unit
				</div>
				<button
					on:click={() => {
						$isInteriorUnitMinimized = !$isInteriorUnitMinimized;
					}}
					class="ghost-btn close-btn !px-0 !py-0"
					id="interior-toggle-amenities"
				>
					{#if !$isInteriorUnitMinimized}
						<img id="mm-am-btn" src={minimizeBtn} alt="minimize" />
					{/if}
					{#if $isInteriorUnitMinimized}
						<img id="mx-am-btn" src={maximizeBtn} alt="maximize" />
					{/if}
				</button>
			</div>

			<div class={!$isInteriorUnitMinimized ? 'block' : 'hidden'}>
				<div class="pt-3">
					<div class="inner-btn-group">
						<button
							class={$hotspotName == 'foyerArea' ? 'active inner-modal-btn ' : 'inner-modal-btn '}
							id="foyerArea-am"
							on:click={() => ($hotspotName = 'foyerArea')}>Foyer Area</button
						>
						<button
							class={$hotspotName == 'living' ? 'active inner-modal-btn ' : 'inner-modal-btn '}
							id="living-am"
							on:click={() => ($hotspotName = 'living')}>Living Room</button
						>

						<button
							class={$hotspotName == 'dining' ? 'active inner-modal-btn ' : 'inner-modal-btn '}
							id="dining-am"
							on:click={() => ($hotspotName = 'dining')}>Dining Area</button
						>

						<button
							class={$hotspotName == 'balcony' ? 'active inner-modal-btn ' : 'inner-modal-btn '}
							id="balcony-am"
							on:click={() => ($hotspotName = 'balcony')}>Balcony</button
						>

						<button
							class={$hotspotName == 'kitchen' ? 'active inner-modal-btn ' : 'inner-modal-btn '}
							id="kitchen-am"
							on:click={() => ($hotspotName = 'kitchen')}>Kitchen</button
						>
						<button
							class={$hotspotName == 'mbedroom' ? 'active inner-modal-btn ' : 'inner-modal-btn '}
							id="mbedroom-am"
							on:click={() => ($hotspotName = 'mbedroom')}>Master Bedroom</button
						>
						<button
							class={$hotspotName == 'washroom1' ? 'active inner-modal-btn ' : 'inner-modal-btn '}
							id="washroom1-am"
							on:click={() => ($hotspotName = 'washroom1')}>Washroom 1</button
						>
						<button
							class={$hotspotName == 'bedroom2' ? 'active inner-modal-btn ' : 'inner-modal-btn'}
							id="bedroom2-am"
							on:click={() => ($hotspotName = 'bedroom2')}
							>Bedroom 2
						</button>
						<button
							class={$hotspotName == 'washroom2' ? 'active inner-modal-btn ' : 'inner-modal-btn '}
							id="washroom2-am"
							on:click={() => ($hotspotName = 'washroom2')}>Washroom 2</button
						>
						<button
							class={$hotspotName == 'bedroom3' ? 'active inner-modal-btn ' : 'inner-modal-btn'}
							id="bedroom3-am"
							on:click={() => ($hotspotName = 'bedroom3')}
							>Bedroom 3
						</button>

						<button
							class={$hotspotName == 'commonWashroom'
								? 'active inner-modal-btn '
								: 'inner-modal-btn '}
							id="commonWashroom-am"
							on:click={() => ($hotspotName = 'commonWashroom')}>Washroom 3</button
						>
					</div>
				</div>
			</div>
		</div>
	{/if}
	{#if $currentbhk == '4bhk'}
		<div class="left-panel p-2">
			<div class="left-panel--header flex justify-between gap-[3rem]">
				<div class="left-title flex items-center font-bold">
					<svg
						width="25"
						height="25"
						viewBox="0 0 32 31"
						class="mr-2"
						fill="none"
						xmlns="http://www.w3.org/2000/svg"
					>
						<path
							d="M15.7834 30.1853C11.9139 30.1853 8.05047 30.1677 4.18106 30.1971C3.40362 30.203 2.89323 29.6084 2.89323 28.9079C2.89917 25.4642 2.89917 22.0205 2.89917 18.5768C2.89917 16.7225 2.89917 14.8682 2.89917 13.0139C2.89917 12.9079 2.95258 12.7666 2.85763 12.7078C2.76267 12.6489 2.67365 12.7784 2.58463 12.8314C2.00897 13.2023 1.4333 13.579 0.851705 13.9499C0.786424 13.9911 0.697404 14.0499 0.643991 14.0323C0.543102 13.997 0.584645 13.8792 0.584645 13.7968C0.584645 12.59 0.584645 11.3833 0.57871 10.1765C0.57871 9.98225 0.649926 9.8704 0.804228 9.77033C2.9051 8.38696 5.00598 7.00359 7.11279 5.62611C8.3294 4.82552 9.54601 4.03082 10.7626 3.23612C12.3412 2.20007 13.9258 1.17578 15.4985 0.133843C15.7181 -0.0133237 15.8902 -0.00743706 16.1098 0.13973C17.5103 1.06982 18.9169 1.98815 20.3234 2.90647C22.8931 4.59594 25.4687 6.28542 28.0385 7.96901C28.9524 8.56945 29.8604 9.16989 30.7803 9.75856C30.9583 9.8704 31.0295 9.99991 31.0236 10.2118C31.0117 11.4009 31.0177 12.5842 31.0177 13.7733C31.0177 13.8616 31.0533 13.9793 30.9761 14.0205C30.8812 14.0794 30.7981 13.9734 30.7209 13.9204C30.1334 13.5437 29.5518 13.1611 28.9643 12.7843C28.899 12.7431 28.8218 12.6371 28.7388 12.696C28.6972 12.7254 28.7031 12.8432 28.7031 12.9197C28.7031 15.6158 28.7031 18.3119 28.7031 21.0139C28.7031 23.6511 28.6913 26.2883 28.7091 28.9315C28.715 29.6202 28.1097 30.1853 27.4391 30.1853C23.5518 30.1735 19.6646 30.1853 15.7834 30.1853ZM15.8071 27.7894C19.1661 27.7894 22.5311 27.7894 25.8901 27.7894C26.2818 27.7894 26.2818 27.7894 26.2818 27.4009C26.2818 22.0382 26.2818 16.6695 26.2877 11.3068C26.2877 11.0889 26.2046 10.9771 26.0325 10.8653C24.6201 9.95282 23.2136 9.0345 21.813 8.11029C19.8842 6.84465 17.9495 5.57313 16.0267 4.30161C15.8605 4.18976 15.7418 4.19565 15.5816 4.3075C14.8219 4.81964 14.0564 5.31412 13.2908 5.81448C10.7092 7.50396 8.12762 9.19932 5.54604 10.8829C5.37393 10.9948 5.32052 11.1184 5.32645 11.3185C5.33239 16.6636 5.33239 22.0087 5.33239 27.3538C5.33239 27.7953 5.33239 27.7953 5.78936 27.7953C9.11871 27.7894 12.4599 27.7894 15.8071 27.7894Z"
							fill="#0F4C5C"
						/>
						<path
							d="M20.4084 26.7112C20.3193 26.7112 20.2303 26.7112 20.1413 26.7112C19.4173 26.7112 19.2036 26.5229 19.1739 25.8223C19.1621 25.5751 19.0137 25.5928 18.8535 25.5928C17.536 25.5928 16.2244 25.5928 14.9069 25.5928C14.1829 25.5928 13.4648 25.5986 12.7407 25.5869C12.5034 25.581 12.4321 25.6634 12.4381 25.893C12.444 26.4404 12.1592 26.7112 11.6013 26.7112C11.2927 26.7112 10.99 26.7171 10.6814 26.7112C10.2601 26.7053 9.93958 26.3992 9.90991 25.9813C9.90991 25.9401 9.90397 25.9048 9.90991 25.8635C9.99299 25.2337 9.8565 24.7098 9.37579 24.2271C8.97223 23.8209 8.82386 23.2499 8.78232 22.6671C8.77045 22.4728 8.74078 22.2786 8.72891 22.0843C8.72297 21.9783 8.65176 21.9371 8.56867 21.89C7.80903 21.4956 7.49449 20.8952 7.60132 20.024C7.67847 19.3705 8.30161 18.7171 9.0019 18.5523C9.95145 18.3345 10.9485 18.9879 11.1443 19.9357C11.2155 20.2712 11.1681 20.6185 11.2571 20.9541C11.4648 21.737 11.8031 22.0078 12.628 22.0078C14.7407 22.0078 16.8594 22.0078 18.9722 22.0078C19.9158 22.0078 20.3015 21.6605 20.4084 20.5714C20.4558 20.1005 20.4855 19.6413 20.7941 19.2469C21.2333 18.6818 21.803 18.4463 22.5033 18.5405C23.1502 18.6288 23.6131 19.0115 23.8742 19.5825C24.2303 20.3654 24.0344 21.4603 23.0374 21.8842C22.8831 21.9489 22.895 22.0725 22.8831 22.202C22.8416 22.9202 22.711 23.6207 22.2422 24.2035C22.1294 24.3389 22.0404 24.5155 21.898 24.6038C21.5953 24.7922 21.5597 25.0571 21.5775 25.3632C21.5894 25.6104 21.5834 25.8518 21.5834 26.099C21.5775 26.4758 21.3579 26.6936 20.9721 26.7053C20.7822 26.7171 20.5923 26.7171 20.4084 26.7112Z"
							fill="#0F4C5C"
						/>
						<path
							d="M15.9302 14.0137C17.1171 14.0137 18.304 14.0255 19.491 14.0137C21.2417 13.9901 22.8797 15.4795 22.945 17.216C22.945 17.2631 22.9509 17.3161 22.9687 17.3632C23.0221 17.5339 22.9806 17.5987 22.7847 17.5692C22.0666 17.4515 21.3841 17.5869 20.7729 17.9695C19.9954 18.4522 19.5919 19.1704 19.5028 20.0711C19.491 20.177 19.4791 20.283 19.4613 20.389C19.3604 20.9482 19.3604 20.9482 18.8026 20.9482C16.7729 20.9482 14.7492 20.9482 12.7195 20.9482C12.3159 20.9482 12.3219 20.9482 12.2803 20.5538C12.2151 19.9298 12.1557 19.3058 11.7581 18.776C11.0815 17.8812 10.2151 17.4044 9.05781 17.5633C8.89164 17.5869 8.84416 17.5339 8.85009 17.3809C8.90351 16.4743 9.24772 15.6855 9.88273 15.0438C10.5059 14.414 11.2714 14.0372 12.1676 14.0196C13.4198 13.996 14.6779 14.0137 15.9302 14.0137Z"
							fill="#0F4C5C"
						/>
					</svg>

					Interior Unit
				</div>
				<button
					on:click={() => {
						$isInteriorUnitMinimized = !$isInteriorUnitMinimized;
					}}
					class="ghost-btn close-btn !px-0 !py-0"
					id="interior-toggle-amenities"
				>
					{#if !$isInteriorUnitMinimized}
						<img id="mm-am-btn" src={minimizeBtn} alt="minimize" />
					{/if}
					{#if $isInteriorUnitMinimized}
						<img id="mx-am-btn" src={maximizeBtn} alt="maximize" />
					{/if}
				</button>
			</div>

			<div class={!$isInteriorUnitMinimized ? 'block' : 'hidden'}>
				<div class="pt-3">
					<div class="inner-btn-group">
						<button
							class={$hotspotName == 'fbhkfoyerspace'
								? 'active inner-modal-btn '
								: 'inner-modal-btn '}
							id="fbhkfoyerspace-am"
							on:click={() => ($hotspotName = 'fbhkfoyerspace')}>Foyer Space</button
						>
						<button
							class={$hotspotName == 'fbhklivingroom'
								? 'active inner-modal-btn '
								: 'inner-modal-btn '}
							id="fbhklivingroom-am"
							on:click={() => ($hotspotName = 'fbhklivingroom')}>Living Room</button
						>
						<button
							class={$hotspotName == 'fbhklivingroom2'
								? 'active inner-modal-btn '
								: 'inner-modal-btn '}
							id="fbhklivingroom2-am"
							on:click={() => ($hotspotName = 'fbhklivingroom2')}>Living Room (view 2)</button
						>
						<button
							class={$hotspotName == 'fbhklivingarea'
								? 'active inner-modal-btn '
								: 'inner-modal-btn '}
							id="fbhklivingarea-am"
							on:click={() => ($hotspotName = 'fbhklivingarea')}>Living Area</button
						>
						<button
							class={$hotspotName == 'fbhkpoojaroom'
								? 'active inner-modal-btn '
								: 'inner-modal-btn '}
							id="fbhkpoojaroom-am"
							on:click={() => ($hotspotName = 'fbhkpoojaroom')}>Pooja Room</button
						>
						<button
							class={$hotspotName == 'fbhklivingbalcony'
								? 'active inner-modal-btn '
								: 'inner-modal-btn '}
							id="fbhklivingbalcony-am"
							on:click={() => ($hotspotName = 'fbhklivingbalcony')}>Living Balcony</button
						>

						<button
							class={$hotspotName == 'fbhkdinningarea'
								? 'active inner-modal-btn '
								: 'inner-modal-btn '}
							id="fbhkdinningarea-am"
							on:click={() => ($hotspotName = 'fbhkdinningarea')}>Dining Area</button
						>

						<button
							class={$hotspotName == 'fbhkkitchen' ? 'active inner-modal-btn ' : 'inner-modal-btn '}
							id="fbhkkitchen-am"
							on:click={() => ($hotspotName = 'fbhkkitchen')}>Kitchen</button
						>

						<button
							class={$hotspotName == 'fbhkutilityspace'
								? 'active inner-modal-btn '
								: 'inner-modal-btn '}
							id="fbhkutilityspace-am"
							on:click={() => ($hotspotName = 'fbhkutilityspace')}>Utility Space</button
						>

						<button
							class={$hotspotName == 'fbhkmasterbedroom'
								? 'active inner-modal-btn '
								: 'inner-modal-btn '}
							id="fbhkmasterbedroom-am"
							on:click={() => ($hotspotName = 'fbhkmasterbedroom')}>Master Bedroom</button
						>

						<button
							class={$hotspotName == 'fbhkmasterbedroom2'
								? 'active inner-modal-btn '
								: 'inner-modal-btn '}
							id="fbhkmasterbedroom2-am"
							on:click={() => ($hotspotName = 'fbhkmasterbedroom2')}>Master Bedroom 2</button
						>

						<button
							class={$hotspotName == 'fbhkwalkinwardrobe'
								? 'active inner-modal-btn '
								: 'inner-modal-btn '}
							id="fbhkwalkinwardrobe-am"
							on:click={() => ($hotspotName = 'fbhkwalkinwardrobe')}>Walk in Wardrobe</button
						>

						<button
							class={$hotspotName == 'fbhkmasterwashroom'
								? 'active inner-modal-btn '
								: 'inner-modal-btn '}
							id="fbhkmasterwashroom-am"
							on:click={() => ($hotspotName = 'fbhkmasterwashroom')}>Master Washroom</button
						>

						<button
							class={$hotspotName == 'fbhkmasterbalcony'
								? 'active inner-modal-btn '
								: 'inner-modal-btn '}
							id="fbhkmasterbalcony-am"
							on:click={() => ($hotspotName = 'fbhkmasterbalcony')}>Master Balcony</button
						>

						<button
							class={$hotspotName == 'fbhkkidsroom1'
								? 'active inner-modal-btn '
								: 'inner-modal-btn '}
							id="fbhkkidsroom1-am"
							on:click={() => ($hotspotName = 'fbhkkidsroom1')}>Kids Room</button
						>

						<button
							class={$hotspotName == 'fbhkkidsroom2'
								? 'active inner-modal-btn '
								: 'inner-modal-btn '}
							id="fbhkkidsroom2-am"
							on:click={() => ($hotspotName = 'fbhkkidsroom2')}>Kids Room 2</button
						>

						<button
							class={$hotspotName == 'fbhkwashroom2'
								? 'active inner-modal-btn '
								: 'inner-modal-btn '}
							id="fbhkwashroom2-am"
							on:click={() => ($hotspotName = 'fbhkwashroom2')}>Washroom 2</button
						>

						<button
							class={$hotspotName == 'fbhkbalcony2'
								? 'active inner-modal-btn '
								: 'inner-modal-btn '}
							id="fbhkbalcony2-am"
							on:click={() => ($hotspotName = 'fbhkbalcony2')}>Balcony 2</button
						>

						<button
							class={$hotspotName == 'fbhkbedroom31'
								? 'active inner-modal-btn '
								: 'inner-modal-btn'}
							id="fbhkbedroom31-am"
							on:click={() => ($hotspotName = 'fbhkbedroom31')}
							>Bedroom 3
						</button>

						<button
							class={$hotspotName == 'fbhkbedroom32'
								? 'active inner-modal-btn '
								: 'inner-modal-btn'}
							id="fbhkbedroom32-am"
							on:click={() => ($hotspotName = 'fbhkbedroom32')}
							>Bedroom 3 (view 2)
						</button>

						<button
							class={$hotspotName == 'fbhkmasterwashroom2'
								? 'active inner-modal-btn '
								: 'inner-modal-btn '}
							id="fbhkmasterwashroom2-am"
							on:click={() => ($hotspotName = 'fbhkmasterwashroom2')}>Master Washroom 2</button
						>

						<button
							class={$hotspotName == 'fbhkmasterbalcony2'
								? 'active inner-modal-btn '
								: 'inner-modal-btn '}
							id="fbhkmasterbalcony2-am"
							on:click={() => ($hotspotName = 'fbhkmasterbalcony2')}>Master Balcony 2</button
						>

						<button
							class={$hotspotName == 'fbhkguestbedroom1'
								? 'active inner-modal-btn '
								: 'inner-modal-btn '}
							id="fbhkguestbedroom1-am"
							on:click={() => ($hotspotName = 'fbhkguestbedroom1')}>Guest bedroom</button
						>

						<button
							class={$hotspotName == 'fbhkguestbedroom2'
								? 'active inner-modal-btn '
								: 'inner-modal-btn'}
							id="fbhkguestbedroom2-am"
							on:click={() => ($hotspotName = 'fbhkguestbedroom2')}
							>Guest Bedroom 2
						</button>

						<button
							class={$hotspotName == 'fbhkwashroom4'
								? 'active inner-modal-btn '
								: 'inner-modal-btn '}
							id="fbhkwashroom4-am"
							on:click={() => ($hotspotName = 'fbhkwashroom4')}>Washrooom 4</button
						>

						<button
							class={$hotspotName == 'fbhkpowderroom'
								? 'active inner-modal-btn '
								: 'inner-modal-btn '}
							id="fbhkpowderroom-am"
							on:click={() => ($hotspotName = 'fbhkpowderroom')}>Powder Room</button
						>
					</div>
				</div>
			</div>
		</div>
	{/if}

	<div class="left-panel p-2">
		<div class="left-panel--header flex justify-between gap-[5rem]">
			<div class="left-title flex items-center font-bold">
				<svg
					width="25"
					height="25"
					viewBox="0 0 32 32"
					fill="none"
					xmlns="http://www.w3.org/2000/svg"
					class="mr-2"
				>
					<path
						d="M8.90798 13.3021C8.90798 10.1749 8.89311 7.0624 8.90798 3.93517C8.92285 2.07654 10.2613 0.527682 12.031 0.262163C13.9196 -0.0328582 15.6596 0.984965 16.2395 2.74034C16.3436 3.06486 16.418 3.40414 16.4477 3.74341C16.5072 4.46621 16.0462 5.02675 15.377 5.11526C14.7375 5.18901 14.2022 4.73173 14.0832 3.99418C13.9345 3.03536 13.414 2.53382 12.5961 2.57808C11.7633 2.62233 11.2874 3.21237 11.2874 4.21544C11.2874 4.76123 11.2725 5.29227 11.2874 5.83806C11.3171 6.47236 11.0197 7.28366 11.4064 7.68194C11.8079 8.09497 12.6258 7.8147 13.2653 7.8147C15.9124 7.82945 18.5743 7.8147 21.2214 7.82945C21.6973 7.82945 21.8758 7.72619 21.8609 7.22466C21.8312 6.11833 21.8312 5.012 21.8609 3.90567C21.9055 1.88478 23.4373 0.321167 25.4598 0.203159C27.4228 0.0999013 29.1033 1.4865 29.3858 3.47789C29.5048 4.33345 29.1033 4.9825 28.3894 5.10051C27.8392 5.18901 27.051 4.70223 27.0064 3.87617C26.9618 3.00586 26.3223 2.50432 25.5341 2.56333C24.7311 2.62233 24.2403 3.21237 24.2403 4.15644C24.2403 10.2781 24.2403 16.3998 24.2403 22.5067C24.2403 22.728 24.2552 22.9493 24.2255 23.1705C24.166 23.8196 23.6901 24.2769 23.0655 24.2916C22.4558 24.3064 21.9501 23.8491 21.8609 23.2148C21.846 23.0968 21.846 22.964 21.846 22.846C21.8312 21.6364 21.8312 21.6364 20.582 21.6364C17.7564 21.6364 14.9309 21.6512 12.1053 21.6217C11.4807 21.6217 11.1684 21.7249 11.2874 22.4182C11.332 22.6543 11.3023 22.905 11.2874 23.1558C11.2428 23.8048 10.7669 24.2769 10.1423 24.2916C9.50283 24.3064 8.96746 23.8343 8.92285 23.1558C8.89311 22.5658 8.90798 21.9757 8.90798 21.3857C8.90798 18.7157 8.90798 16.0015 8.90798 13.3021ZM16.6113 13.8184C18.1431 13.8184 19.6748 13.8036 21.2214 13.8331C21.7568 13.8479 21.9055 13.6709 21.8758 13.1693C21.8312 12.358 21.8609 11.5467 21.8758 10.7354C21.8907 10.3814 21.8014 10.2044 21.385 10.2044C18.1877 10.2191 15.0052 10.2191 11.8079 10.2044C11.4064 10.2044 11.3023 10.3519 11.3023 10.7207C11.3171 11.5615 11.332 12.3875 11.3023 13.2283C11.2725 13.7299 11.4807 13.8479 11.9417 13.8331C13.4883 13.8036 15.0498 13.8184 16.6113 13.8184ZM16.6113 19.3058C18.1728 19.3058 19.7343 19.291 21.2958 19.3205C21.7568 19.3353 21.8907 19.1878 21.8758 18.7452C21.846 18.0519 21.8609 17.3734 21.8758 16.6801C21.8758 16.3556 21.8163 16.1933 21.4296 16.1933C18.2174 16.208 14.9904 16.208 11.7781 16.1933C11.4212 16.1933 11.3023 16.3113 11.3171 16.6653C11.332 17.3586 11.3617 18.0372 11.3171 18.7305C11.2874 19.2468 11.4956 19.3353 11.9566 19.3353C13.4883 19.291 15.0498 19.3058 16.6113 19.3058Z"
						fill="#0F4C5C"
					/>
					<path
						d="M25.8935 31.0057C24.6808 31.0057 23.661 30.5336 22.8204 29.6043C22.4483 29.206 22.1038 28.7635 21.5664 28.6013C20.6568 28.3357 19.954 28.7193 19.3615 29.4421C18.5208 30.4451 17.4873 30.9909 16.2195 31.0057C14.9792 31.0204 13.9594 30.5041 13.0913 29.5748C11.7683 28.144 10.9001 28.144 9.59098 29.5601C7.77193 31.522 5.20873 31.5367 3.40347 29.5748C2.87981 28.9995 2.32858 28.5423 1.5293 28.4832C0.867833 28.439 0.481975 27.849 0.550878 27.2147C0.619782 26.5656 1.08832 26.1378 1.74979 26.1673C2.93493 26.2116 3.92714 26.7131 4.74019 27.6129C5.15361 28.0702 5.55325 28.5275 6.21472 28.616C7.09668 28.734 7.68925 28.2472 8.24048 27.6129C9.27402 26.4329 10.5556 25.9756 12.0439 26.2263C12.981 26.3886 13.7665 26.8754 14.4142 27.5982C14.8414 28.0702 15.2686 28.5423 15.9576 28.616C16.8258 28.7045 17.377 28.2177 17.9145 27.6129C19.1685 26.2263 20.6706 25.8281 22.3794 26.3886C23.096 26.6246 23.661 27.1114 24.1847 27.6867C25.3836 28.9995 26.3345 28.9995 27.5334 27.6719C28.3464 26.7574 29.3249 26.2263 30.4962 26.1526C31.2128 26.1083 31.7089 26.5214 31.7778 27.1852C31.8467 27.8785 31.4609 28.4242 30.758 28.4832C30.0139 28.5423 29.4764 28.9405 28.9941 29.5011C28.1673 30.4894 27.1337 30.9909 25.8935 31.0057Z"
						fill="#0F4C5C"
					/>
				</svg>
				Details
			</div>
			<button
				on:click={() => {
					$isInteriorUnitDataMinimized = !$isInteriorUnitDataMinimized;
				}}
				id="minimize-interior-data-amenities"
				class="ghost-btn close-btn !px-0 !py-0"
			>
				{#if !$isInteriorUnitDataMinimized}
					<img id="un-mm" src={minimizeBtn} alt="minimize" />
				{/if}
				{#if $isInteriorUnitDataMinimized}
					<img id="un-mx" src={maximizeBtn} alt="maximize" />
				{/if}
			</button>
		</div>

		<div class={!$isInteriorUnitDataMinimized ? 'block' : 'hidden'}>
			<div class="no-hovers pt-3">
				<div class="inner-btn-group">
					<div class="inner-modal-btn" id="int1">
						{#if $currentbhk == '3bhk'}
							<span>Unit 3 of Tower 2 : </span>
							<span class="ml-auto">1214 Sqft</span>
						{/if}
						{#if $currentbhk == '4bhk'}
							<span>Altair Unit no.1 : </span>
							<span class="ml-auto">1880.47sqft</span>
						{/if}
					</div>
				</div>
			</div>
		</div>
	</div>
</div>

{#if $currentbhk == '4bhk' && !['fbhkbalcony2', 'fbhkdinningarea', 'fbhkguestbedroom2', 'fbhkkitchen', 'fbhklivingroom', 'fbhkmasterbalcony2', 'fbhkmasterwashroom', 'fbhkpowderroom', 'fbhkwashroom2', 'fbhkbedroom31', 'fbhkfoyerspace', 'fbhkkidsroom1', 'fbhklivingarea', 'fbhklivingroom2', 'fbhkmasterbedroom', 'fbhkmasterwashroom2', 'fbhkutilityspace', 'fbhkwashroom4', 'fbhkbedroom32', 'fbhkguestbedroom1', 'fbhkkidsroom2', 'fbhklivingbalcony', 'fbhkmasterbalcony', 'fbhkmasterbedroom2', 'fbhkpoojaroom', 'fbhkwalkinwardrobe'].includes($hotspotName)}
	<div class="dollhouse-wrapper z-[9]" transition:fade>
		<img src={fbhkdollhouse} id="dollhouse" class="pointer z-[9000]" alt="dollhouse" />
		<button
			id="start-tour"
			class="primary-btn mx-auto mt-[-1rem]"
			data-vretail-product-id="4bhk-start-tour" data-vretail-name="start tour"
			on:click={() => ($hotspotName = 'fbhkfoyerspace')}
		>
			<img id="startTourImg" src={startTourImg} alt="start tour" />
			Start Tour
		</button>
	</div>
{/if}
<div id="pano"></div>
<div id="spot"></div>
