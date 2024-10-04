<script lang="ts">
	import { onMount, onDestroy } from 'svelte';
	import { browser } from '$app/environment';
	import { EXRLoader } from 'three/examples/jsm/loaders/EXRLoader.js';
	import type { Vector3 } from 'three';

	let container: HTMLDivElement | null = null;
	let loading = true;
	let loading_percentage = 0;
	let planet_created = false;
	export let planet;
	export let three;
	export let is_comparing;
	const THREE = three;
	export let scene: any;
	export let camera: any;
	let earth_data: any[] = [];

	if (browser) {
		let renderer: any;
		let pmremGenerator: any;
		let isUserInteracting = false;
		let previousMousePosition = { x: 0, y: 0 };
		let rotation = { x: 0, y: 0 };
		let raycaster = new THREE.Raycaster();
		let mouse = new THREE.Vector2();
		let glow_material: any;
		let planeted: any;

		const init = () => {
			scene = new THREE.Scene();

			camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
			camera.position.set(0, 0, 100);
			camera.lookAt(0, 0, 0);

			renderer = new THREE.WebGLRenderer({ antialias: true });
			renderer.setSize(window.innerWidth, window.innerHeight);

			renderer.shadowMap.enabled = true;

			renderer.setPixelRatio(window.devicePixelRatio);
			if (container) {
				container.appendChild(renderer.domElement);
			}

			renderer.toneMapping = THREE.ACESFilmicToneMapping;

			pmremGenerator = new THREE.PMREMGenerator(renderer);
			pmremGenerator.compileEquirectangularShader();

			const loader = new EXRLoader();
			loader.load(
				'https://sapphire-voluntary-gecko-251.mypinata.cloud/ipfs/QmZv3Z6YC8yKLz2AWrVYgGmVWEwbwGSSXRTBjFxiCQ7uXx',
				(texture, _) => {
					const envMap = pmremGenerator.fromEquirectangular(texture).texture;

					scene.background = envMap;
					scene.backgroundIntensity = 0.05;
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
			if (planet_created) return;
			planet_created = true;
			const geometry = new THREE.SphereGeometry(20 * 2.3, 64, 64);
			const textureLoader = new THREE.TextureLoader();
			const texture = textureLoader.load(planet.resource);

			const material = new THREE.MeshStandardMaterial({
				map: texture,
				metalness: 0.1,
				roughness: 0.5
			});

			planeted = new THREE.Mesh(geometry, material);
			planeted.position.set(-25, 0, 0);
			planeted.receiveShadow = true;
			scene.add(planeted);
			addEarth(handleEarthMassCompared(planet.nasa_data.radius_earth_compared));
			
			const directionalLight = new THREE.DirectionalLight(0xffffff, 1.2);
			directionalLight.position.set(20, 0, 7.5);
			directionalLight.castShadow = true;
			scene.add(directionalLight);

			const ambientLight = new THREE.AmbientLight(0x404040, 0.5);
			scene.add(ambientLight);

			const pointLight = new THREE.PointLight(0xffffff, 2.5, 50);
			pointLight.position.set(planeted.position.x, planeted.position.y, planeted.position.z);
			scene.add(pointLight);

			const glowGeometry = new THREE.SphereGeometry((20 * 2.34) , 64, 64);
			const glowMaterial = new THREE.MeshBasicMaterial({
				color: planet.color,
				transparent: true,
				opacity: 0.1,
				blending: THREE.AdditiveBlending,
				depthWrite: false
			});
			planeted.rotation.x += 0.5;
			const glowMesh = new THREE.Mesh(glowGeometry, glowMaterial);
			glowMesh.castShadow = true;
			glowMesh.scale.set(planeted.scale.x, planeted.scale.y, planeted.scale.z);
			glowMesh.position.set(planeted.position.x, planeted.position.y, planeted.position.z);
			scene.add(glowMesh);
			glow_material = glowMesh;
		};

		const handleEarthMassCompared = (comparison_times: string) => {
			if (comparison_times.includes('+')) {
				return (parseFloat(comparison_times.split('+')[0]));
			}

			if (comparison_times.includes('<')) {
				return (parseFloat(comparison_times.split('<')[1]));
			}
			if (comparison_times.includes('±')) {
				return (parseFloat(comparison_times.split('±')[0]));
			}
			return 1;
		}

		const addEarth = (comparison_times: number) => {
			let geometry;
			if (comparison_times < 1) {
				planeted.scale.set(comparison_times, comparison_times, comparison_times);
				geometry = new THREE.SphereGeometry((20 * 2.3) , 64, 64);
		} else {
				geometry = new THREE.SphereGeometry((20 * 2.3) / comparison_times , 64, 64);
			}
			
			const textureLoader = new THREE.TextureLoader();
			const texture = textureLoader.load("/earth.jpg");
			
			const material = new THREE.MeshStandardMaterial({
				map: texture,
				metalness: 0.1,
				roughness: 0.5,
				color: new THREE.Color(0xaaaaaa),
			});

			let planet = new THREE.Mesh(geometry, material);
			planet.position.set(-40, 0, 0);
			planet.receiveShadow = true;

			const directionalLight = new THREE.DirectionalLight(0xffffff, 1.2);
			directionalLight.position.set(20, 0, 7.5);
			directionalLight.castShadow = true;

			const ambientLight = new THREE.AmbientLight(0x404040, 0.5);

			const pointLight = new THREE.PointLight(0xffffff, 2.5, 50);
			pointLight.position.set(planet.position.x, planet.position.y, planet.position.z);
			scene.add(pointLight);
			planet.rotation.x += 0.5;

			earth_data.push(planet);
			earth_data.push(pointLight);
			earth_data.push(ambientLight);
			earth_data.push(directionalLight);

		}


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
				if (planeted) {
					planeted.rotation.x = rotation.x;
					planeted.rotation.y = rotation.y;
				}

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
				if (planeted) {
					planeted.rotation.y = rotation.y;
					planeted.rotation.x = rotation.x;
				}

				previousMousePosition = { x: event.touches[0].pageX, y: event.touches[0].pageY };
			}
		};

		const onTouchEnd = () => {
			isUserInteracting = false;
		};

		const animate = () => {
			requestAnimationFrame(animate);
			if (planeted) {
			if (!is_comparing) {
				
				if (planeted.position && planeted.position.x != -25) {
					earth_data.forEach((data) => {
						scene.remove(data);
					});
					planeted.position.set(-25, 0, 0);
					if (glow_material) glow_material.position.set(planeted.position.x, planeted.position.y, planeted.position.z);
				}
			}
			else {
				if (planeted.position && planeted.position.x != 60) {
					while (planeted.position.x != 60) {
						earth_data.forEach((data: any) => {
							scene.add(data);
						});
						planeted.position.set(60, 0, 0);
						if (glow_material) glow_material.position.set(planeted.position.x, planeted.position.y, planeted.position.z);
					}
				}
			}
			}
			if (planeted) planeted.rotation.y += 0.0005;
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
			if (scene) scene = null;
			if (camera) camera = null;
			if (renderer) renderer = null;
			if (planeted) planeted = null;
		});
	}
</script>

{#if loading}
	<p>Loading WebGL... ({loading_percentage == 0 ? 0 : loading_percentage}%)</p>
{/if}
<div bind:this={container} style="visibility: {loading ? 'hidden' : 'visible'};"></div>


<style >
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
