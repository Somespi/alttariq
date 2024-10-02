<script lanf="ts">
    import PlanetFromModel from "./PlanetFromModel.svelte";
    import Planet from "./Planet.svelte";
    export let planet; 
    
</script>

{#if planet.ismodel}
    <PlanetFromModel planet={planet} />
{:else}
    <Planet planet={planet} />
{/if}