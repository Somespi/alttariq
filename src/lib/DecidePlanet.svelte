<script lang="ts">
	import PlanetFromModel from './PlanetFromModel.svelte';
	import Planet from './Planet.svelte';
	export let THREE;
	export let scene;
	export let camera;
	export let planet;
	export let is_comparing;
</script>

{#if planet.ismodel}
	<PlanetFromModel {is_comparing} {planet} three={THREE} {scene} {camera} />
{:else}
	<Planet {is_comparing} {planet} three={THREE} {scene} {camera} />
{/if}
