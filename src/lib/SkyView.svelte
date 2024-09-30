<script lang="ts">
	import { onMount, onDestroy } from 'svelte';
	import * as THREE from 'three';
	import { browser } from '$app/environment';
	import { EXRLoader } from 'three/examples/jsm/loaders/EXRLoader.js';
	import { FontLoader } from 'three/examples/jsm/Addons.js';
	import { TextGeometry } from 'three/addons/geometries/TextGeometry.js';
	import { SpriteText2D, textAlign } from 'three-text2d'
	import { Text2D } from 'three-text2d/lib/Text2D';

	let container: HTMLDivElement | null = null;
	let loading = true;
    let loading_percentage = 0;
	export let planets: {id: number, x: number, y: number, z: number, color: number, name: string}[] = [];
	export let onPlanetClick: (id: number) => void = (id) => {};

	if (browser) {
		let camera: THREE.PerspectiveCamera;
		let scene: THREE.Scene;
		let renderer: THREE.WebGLRenderer;
		let pmremGenerator: THREE.PMREMGenerator;
		let isUserInteracting = false;
		let lon = 0, lat = 0;
		let raycaster = new THREE.Raycaster();
		let mouse = new THREE.Vector2();
		let onPointerDownMouseX = 0, onPointerDownMouseY = 0;
		let onPointerDownLon = 0, onPointerDownLat = 0;
        let planetsArr: THREE.Mesh[] = [];
		



		const init = () => {
			scene = new THREE.Scene();

			camera = new THREE.PerspectiveCamera(
				75,
				window.innerWidth / window.innerHeight,
				0.1,
				1000
			);

			camera.position.set(0,0,0);
			camera.lookAt(250, -120, -250);

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
				},
				(xhr) => {
					console.log(`Loading texture: ${Math.round((xhr.loaded / xhr.total) * 100)}%`);
                    loading_percentage = Math.round((xhr.loaded / xhr.total) * 100);
                    createPlanets();
				},
				(error) => {
					console.error('Error loading texture:', error);
				}
			);

			document.addEventListener('mousedown', onPointerDown, false);
			document.addEventListener('mousemove', onPointerMove, false);
			document.addEventListener('mouseup', onPointerUp, false);
			document.addEventListener('click', onDocumentClick, false);
			document.addEventListener('touchstart', onTouchStart, false);
			document.addEventListener('touchmove', onTouchMove, false);
			document.addEventListener('touchend', onTouchEnd, false);

			window.addEventListener('resize', onWindowResize, false);
		};

		const onDocumentClick = (event: MouseEvent) => {
			mouse.x = (event.clientX / window.innerWidth) * 2 - 1;
			mouse.y = -(event.clientY / window.innerHeight) * 2 + 1;

			raycaster.setFromCamera(mouse, camera);
			const intersects = raycaster.intersectObjects(planetsArr);

			if (intersects.length > 0) {
				const clickedPlanet = intersects[0].object;
				onPlanetClick(planetsArr.findIndex(planet => planet === clickedPlanet));
			}
		};

	


        const createPlanets = async () => {
			const planetPositions = planets;
            
			const planetGeometry = new THREE.SphereGeometry(20, 25, 25);
            
            
			planetPositions.forEach(async (planetprops) =>  {
				const planet = new THREE.Mesh(planetGeometry, new THREE.MeshStandardMaterial({ color: planetprops.color , emissive: planetprops.color, emissiveIntensity: 12}));
				planet.position.set(planetprops.x, planetprops.y, planetprops.z);
				// var sprite =  Text2D(`Planet ${planetprops.id}`, { align: textAlign.center,  font: '40px Arial', fillStyle: '#ffffff' , antialias: false })
				// sprite.position.set(planetprops.x, planetprops.y, planetprops.z);
				// scene.add(sprite)
				scene.add(planet)
				planetsArr.push(planet); 
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
			console.log(target)
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
	<p>Loading WebGL... ({loading_percentage == 0 ? 0 : loading_percentage - 1}%)</p>
{/if}
<div
	bind:this={container}
	style="visibility: {loading ? 'hidden' : 'visible'};"
	class="relative h-screen w-screen"
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
</style>
