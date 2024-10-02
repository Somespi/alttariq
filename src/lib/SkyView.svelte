<script lang="ts">
	import { onMount, onDestroy } from 'svelte';
	import { browser } from '$app/environment';
	import { EXRLoader } from 'three/examples/jsm/loaders/EXRLoader.js';
	import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader.js';
	import { FontLoader } from 'three/examples/jsm/loaders/FontLoader.js';
	import { TextGeometry } from 'three/examples/jsm/geometries/TextGeometry.js';
	import { MeshBasicMaterial } from 'three/src/materials/MeshBasicMaterial.js';
	import { Mesh } from 'three/src/objects/Mesh.js';
	import * as THREE from 'three';
	import { fade } from 'svelte/transition';

	let container: HTMLDivElement | null = null;
	let loading = true;
	let loading_percentage = 0;
	export let planets: any[] = [];
	export let onPlanetClick: (id: number) => void = (id) => {};
	let three;
	export let scene: any;
	export let camera: any;

	if (browser) {
		let renderer: THREE.WebGLRenderer;
		let pmremGenerator: THREE.PMREMGenerator;
		let isUserInteracting = false;
		let lon = 0, lat = 0;
		let raycaster = new THREE.Raycaster();
		let mouse = new THREE.Vector2();
		let onPointerDownMouseX = 0,
			onPointerDownMouseY = 0;
		let onPointerDownLon = 0,
			onPointerDownLat = 0;
		let planetsArr: any[] = [];

		const init = () => {
			scene = new THREE.Scene();
			camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
			camera.position.set(0, 0, 0);
			camera.lookAt(250, -120, -250);

			renderer = new THREE.WebGLRenderer({ antialias: true });
			renderer.setSize(window.innerWidth, window.innerHeight);
			renderer.setPixelRatio(window.devicePixelRatio);
			if (container) container.appendChild(renderer.domElement);
			renderer.shadowMap.enabled = true; 
            renderer.shadowMap.type = THREE.PCFSoftShadowMap;

			pmremGenerator = new THREE.PMREMGenerator(renderer);
			pmremGenerator.compileEquirectangularShader();

			const loader = new EXRLoader();
			loader.load(
				'/skybox/Milkyway.exr',
				(texture) => {
					const envMap = pmremGenerator.fromEquirectangular(texture).texture;
					scene.background = scene.environment = envMap;
					scene.backgroundIntensity = 0.03;
					texture.dispose();
					createPlanets();
					loading = false;
				},
				(xhr) => {
					loading_percentage = Math.round((xhr.loaded / xhr.total) * 100);
				},
				(error) => {
					console.error('Error loading texture:', error);
				}
			);

			const ambientLight = new THREE.AmbientLight(0x404040, 1); 
            scene.add(ambientLight);

			addEventListeners();
		};

		const addEventListeners = () => {
			document.addEventListener('mousedown', onPointerDown);
			document.addEventListener('mousemove', onPointerMove);
			document.addEventListener('mouseup', onPointerUp);
			document.addEventListener('click', onDocumentClick);
			document.addEventListener('touchstart', onTouchStart);
			document.addEventListener('touchmove', onTouchMove);
			document.addEventListener('touchend', onTouchEnd);
			window.addEventListener('resize', onWindowResize);
		};

		const onDocumentClick = (event: MouseEvent) => {
	mouse.x = (event.clientX / window.innerWidth) * 2 - 1;
	mouse.y = -(event.clientY / window.innerHeight) * 2 + 1;

	raycaster.setFromCamera(mouse, camera);
	const intersects = raycaster.intersectObjects(planetsArr, true); 

	if (intersects.length > 0) {
		let clickedObject = intersects[0].object;
		let planetIndex = planetsArr.indexOf(clickedObject);

		if (planetIndex === -1 && clickedObject.parent) {
			planetIndex = planetsArr.indexOf(clickedObject.parent);
		}

		if (planetIndex !== -1) {
			onPlanetClick(planetIndex);
		}
	}
};


		const createPlanets = async () => {
			if (planetsArr.length > 0) return; 
			const planetGeometry = new THREE.SphereGeometry(20, 25, 25);
			const textureLoader = new THREE.TextureLoader();
			const gltfLoader = new GLTFLoader();

			for (const planetprops of planets) {
				let planet;

				if (planetprops.ismodel) {
					const gltf = await gltfLoader.loadAsync(`/models/${planetprops.name}.glb`);
					planet = gltf.scene;
					planet.scale.set(0.05, 0.05, 0.05);
				} else {
					const texture = textureLoader.load(`/planets/${planetprops.name}.jpg`);
					const material = new THREE.MeshStandardMaterial({ map: texture, metalness: 0.1, roughness: 0.5 });
					planet = new THREE.Mesh(planetGeometry, material);
				}

				planet.position.set(planetprops.x, planetprops.y, planetprops.z);
				planet.receiveShadow = true;
				scene.add(planet);
				planetsArr.push(planet);
				
				addLabel(planetprops.name, planet.position);



				const planetLight = new THREE.PointLight(0xffffff, 1.5, 75);
				planetLight.position.copy(planet.position);
				planetLight.castShadow = true;
				scene.add(planetLight);
			}
		};

		const fontLoader = new FontLoader();
		const addLabel = (text: string, position: { x: number; y: number; z: number; }) => {
			fontLoader.load('https://threejs.org/examples/fonts/helvetiker_regular.typeface.json', (font) => {
				const textGeometry = new TextGeometry(text, { font, size: 6, depth: 1, bevelEnabled: false });
				const textMesh = new Mesh(textGeometry, new MeshBasicMaterial({ color: 0xffffff }));
				textMesh.position.set(position.x, position.y - 40, position.z - 20);
				textMesh.renderOrder = 1;
				textMesh.lookAt(camera.position);
				scene.add(textMesh);
			});
		};

		const onWindowResize = () => {
			camera.aspect = window.innerWidth / window.innerHeight;
			camera.updateProjectionMatrix();
			renderer.setSize(window.innerWidth, window.innerHeight);
		};

		const onPointerDown = (event: MouseEvent) => {
			isUserInteracting = true;
			onPointerDownMouseX = event.clientX;
			onPointerDownMouseY = event.clientY;
			onPointerDownLon = lon;
			onPointerDownLat = lat;
		};

		const onPointerMove = (event: MouseEvent) => {
			if (isUserInteracting) {
				lon = (onPointerDownMouseX - event.clientX) * 0.1 + onPointerDownLon;
				lat = (event.clientY - onPointerDownMouseY) * 0.1 + onPointerDownLat;
			}
		};

		const onPointerUp = () => {
			isUserInteracting = false;
		};

		const onTouchStart = (event: TouchEvent) => {
			if (event.touches.length === 1) {
				isUserInteracting = true;
				onPointerDownMouseX = event.touches[0].pageX;
				onPointerDownMouseY = event.touches[0].pageY;
				onPointerDownLon = lon;
				onPointerDownLat = lat;
			}
		};

		const onTouchMove = (event: TouchEvent) => {
			if (isUserInteracting && event.touches.length === 1) {
				lon = (onPointerDownMouseX - event.touches[0].pageX) * 0.1 + onPointerDownLon;
				lat = (event.touches[0].pageY - onPointerDownMouseY) * 0.1 + onPointerDownLat;
			}
		};

		const onTouchEnd = () => {
			isUserInteracting = false;
		};

		const animate = () => {
			requestAnimationFrame(animate);
			update();
		};

		const update = () => {
			if (!isUserInteracting) {
				lon += 0.02;
			}

			lat = Math.max(-85, Math.min(85, lat));
			const phi = THREE.MathUtils.degToRad(90 - lat);
			const theta = THREE.MathUtils.degToRad(lon);

			const target = new THREE.Vector3(
				500 * Math.sin(phi) * Math.cos(theta),
				500 * Math.cos(phi),
				500 * Math.sin(phi) * Math.sin(theta)
			);

			camera.lookAt(target);
			renderer.render(scene, camera);
		};

		onMount(() => {
			init();
			animate();
		});

		onDestroy(() => {
			document.removeEventListener('mousedown', onPointerDown);
			document.removeEventListener('mousemove', onPointerMove);
			document.removeEventListener('mouseup', onPointerUp);

			document.removeEventListener('touchstart', onTouchStart);
			document.removeEventListener('touchmove', onTouchMove);
			document.removeEventListener('touchend', onTouchEnd);

			window.removeEventListener('resize', onWindowResize);

			if (renderer) renderer.dispose();
			if (pmremGenerator) pmremGenerator.dispose();
		});
	}
</script>

{#if loading}
	<p>Loading WebGL... ({loading_percentage === 0 ? 0 : loading_percentage - 1}%)</p>
{/if}
<div
	transition:fade
	bind:this={container}
	style="visibility: {loading ? 'hidden' : 'visible'};"
	class="relative h-screen w-screen"
></div>

<style>
	html, body {
		margin: 0;
		padding: 0;
		overflow: hidden;
	}
</style>
