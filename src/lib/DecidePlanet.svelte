<script lang="ts">
	import PlanetFromModel from './PlanetFromModel.svelte';
	import Planet from './Planet.svelte';
	export let THREE;
	export let scene;
	export let camera;
	export let planet;

	// let scene = new THREE.Scene();
	// let camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
</script>

{#if planet.ismodel}
	<PlanetFromModel {planet} three={THREE} {scene} {camera} />
{:else}
	<Planet {planet} three={THREE} {scene} {camera} />
{/if}
