<script lang="ts">
	import { onMount, onDestroy } from 'svelte';
	import * as THREE from 'three';
	import { browser } from '$app/environment';
	import { EXRLoader } from 'three/examples/jsm/loaders/EXRLoader.js';

	let container: HTMLDivElement | null = null;
	let loading = true;
	let loading_percentage = 0;
	export let planet;

	if (browser) {
		let camera: THREE.PerspectiveCamera;
		let scene: THREE.Scene;
		let renderer: THREE.WebGLRenderer;
		let pmremGenerator: THREE.PMREMGenerator;
		let isUserInteracting = false;
		let previousMousePosition = { x: 0, y: 0 };
		let rotation = { x: 0, y: 0 };
		let raycaster = new THREE.Raycaster();
		let mouse = new THREE.Vector2();

		let planet: THREE.Mesh;

		const init = () => {
			scene = new THREE.Scene();

			camera = new THREE.PerspectiveCamera(
				75,
				window.innerWidth / window.innerHeight,
				0.1,
				1000
			);
			camera.position.set(0, 0, 100);
			camera.lookAt(0, 0, 0);

			renderer = new THREE.WebGLRenderer({ antialias: true });
			renderer.setSize(window.innerWidth, window.innerHeight);
			
			renderer.setPixelRatio(window.devicePixelRatio);
			if (container) {
				container.appendChild(renderer.domElement);
			}

			pmremGenerator = new THREE.PMREMGenerator(renderer);
			pmremGenerator.compileEquirectangularShader();

			const loader = new EXRLoader();
			loader.load(
				"/starmap.exr",
				(texture, _) => {
					const envMap = pmremGenerator.fromEquirectangular(texture).texture;
					scene.background = envMap;
					scene.environment = envMap;
					texture.dispose();
					loading = false;
					createPlanet(); 
				},
				(xhr) => {
					loading_percentage = Math.round((xhr.loaded / xhr.total) * 100);
				},
				(error) => {
					console.error('Error loading texture:', error);
				}
			);

			document.addEventListener('mousedown', onPointerDown, false);
			document.addEventListener('mousemove', onPointerMove, false);
			document.addEventListener('mouseup', onPointerUp, false);
			document.addEventListener('touchstart', onTouchStart, false);
			document.addEventListener('touchmove', onTouchMove, false);
			document.addEventListener('touchend', onTouchEnd, false);
			window.addEventListener('resize', onWindowResize, false);
		};

		const createPlanet = () => {
			const geometry = new THREE.SphereGeometry(20, 64, 64);
			const material = new THREE.MeshStandardMaterial({ color: 0x0077ff });
			planet = new THREE.Mesh(geometry, material);
			scene.add(planet);
		};

		const onWindowResize = () => {
			camera.aspect = window.innerWidth / window.innerHeight;
			camera.updateProjectionMatrix();
			renderer.setSize(window.innerWidth, window.innerHeight);
		};

		const onPointerDown = (event: MouseEvent) => {
			isUserInteracting = true;
			previousMousePosition = { x: event.clientX, y: event.clientY };
		};

		const onPointerMove = (event: MouseEvent) => {
			if (isUserInteracting && planet) {
				const deltaMove = {
					x: event.clientX - previousMousePosition.x,
					y: event.clientY - previousMousePosition.y
				};

				rotation.y += deltaMove.x * 0.005;
				rotation.x += deltaMove.y * 0.005;

				planet.rotation.y = rotation.y;
				planet.rotation.x = rotation.x;

				previousMousePosition = { x: event.clientX, y: event.clientY };
			}
		};

		const onPointerUp = () => {
			isUserInteracting = false;
		};

		const onTouchStart = (event: TouchEvent) => {
			if (event.touches.length === 1) {
				isUserInteracting = true;
				previousMousePosition = { x: event.touches[0].pageX, y: event.touches[0].pageY };
			}
		};

		const onTouchMove = (event: TouchEvent) => {
			if (isUserInteracting && event.touches.length === 1 && planet) {
				const deltaMove = {
					x: event.touches[0].pageX - previousMousePosition.x,
					y: event.touches[0].pageY - previousMousePosition.y
				};

				rotation.y += deltaMove.x * 0.005;
				rotation.x += deltaMove.y * 0.005;

				planet.rotation.y = rotation.y;
				planet.rotation.x = rotation.x;

				previousMousePosition = { x: event.touches[0].pageX, y: event.touches[0].pageY };
			}
		};

		const onTouchEnd = () => {
			isUserInteracting = false;
		};

		const animate = () => {
			requestAnimationFrame(animate);
			render();
		};

		const render = () => {
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
	<p>Loading WebGL... ({loading_percentage == 0 ? 0 : loading_percentage}%)</p>
{/if}
<div
	bind:this={container}
	style="visibility: {loading ? 'hidden' : 'visible'};"
></div>

<style>
	html,
	body {
		margin: 0;
		padding: 0;
		overflow: hidden;
		height: 100%;
		width: 100%;
	}
	p {
		position: absolute;
		top: 50%;
		left: 50%;
		transform: translate(-50%, -50%);
		font-size: 1.5rem;
		color: #ffffff;
	}
</style>
