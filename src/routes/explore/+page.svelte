<script lang="ts"> 
    import SkyView from '$lib/SkyView.svelte';
	import { onMount } from 'svelte';
	$: id = -1;
	$: is_loaded = false;
	$: overview = '';
	$: details = '';

    const planets =  [
				{id: 0, x: 500, y: 10, z: 9 , color: 0xffcc00, name: 'Kepler-22b'},
				{id: 1, x: 382, y: -176, z: 23, color: 0xffcc00, name: 'Kepler-62b'},
				{id: 2, x: 150, y: -200, z: -500, color: 0xffcc00, name: 'Kepler-62c'},
			];

	onMount(() => {
		let url = new URL(window.location.href);
		let planet_id = url.searchParams.get('id');
		if (planet_id) {
			id = parseInt(planet_id);
			
		}
		is_loaded = true;
		if (is_loaded && id != -1) {
			onPlanetClick(id);
		}
	})

	const onPlanetClick = (planet_id: number) => {
		let url = new URL(window.location.href);
		url.searchParams.set('id', planet_id.toString());
		window.history.replaceState(null, '', url);
		id = planet_id;

		fetch("http://localhost:8080" + '/prompt', {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json'
                },
                body: JSON.stringify({
                    "prompt": "Generate an overview for the given planet: " + planets[planet_id].name + ", and make sure to best-fit the content with the: " + sessionStorage.getItem('grade') + " educational grade.",
                    "command": 
                    "You are going to act as only-education tailor, your primary goal is to generate an overview description of the given planet, for example: \
					if you were asked to generate an overview for the planet : Kepler-22 b, then you will respond with something like: A potentially rocky world, larger than Earth and Super Earth, A possible ocean world orbiting in the habitable zone—the region around a star where the temperature is right for liquid water, a requirement for life on Earth\
					.\nHighlighting the given planet in the overview: Kepler-22 b with its type and size, and its key features is embedded between <b> and </b> to highlight it.\
					Also, make sure to best-fit the content with the given educational grade.",
                    "message": "Ok, I'm going to act and behave as described and generate the overview to best-fit the given educational grade."
                })
            }).then(res => res.json())
			.then(data => {
					overview = data['ai_model']
					console.log(data['ai_model'])
					console.log(data)
			})


			fetch("http://localhost:8080" + '/prompt', {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json'
                },
                body: JSON.stringify({
                    "prompt": "Generate a detailed description for the given planet: " + planets[planet_id].name + ", and make sure to best-fit the content with the: " + sessionStorage.getItem('grade') + " educational grade.",
                    "command": 
                    "You are going to act as only-education tailor, your primary goal is to generate a detailed description of the given planet, for example: \
					if you were asked to generate an overview for the planet : Kepler-22 b, then you will respond with something that includes its properties such as gravity, mass and other primary things, don't forget to mention the numbers and use the information that i am going to give you which are from NASA to authoritise your work. \
					.\nHighlighting the given planet properties in the details with its type and size, and its key features is embedded between <b> and </b> to highlight it. (use html)\
					Also, make sure to best-fit the content with the given educational grade.",
                    "message": "Ok, I'm going to act and behave as described and generate the overview to best-fit the given educational grade with using html for formatting the highlighting."
                })
            }).then(res => res.json())
			.then(data => {
					details = data['ai_model']
					console.log(data['ai_model'])
					console.log(data)
			})
	}
</script>
{#if !is_loaded}
Loading...
{:else if id == -1}
<SkyView {planets} {onPlanetClick}/>
{:else}
	<div class="flex h-screen w-full flex-row">
		<div class=" w-[63%] bg-slate-950 h-full">
			test
		</div>

		<div class="flex-1 h-full p-3">
			<div class="p-3 bg-accent rounded-2xl w-full h-full text-accent-content overflow-y-auto">
				<h1 class="text-lg font-medium">
					Planet Name
				</h1>

				<p class="text-xs font-bold mt-12">
					Planet Overview
				</p>
				<p id="small_desc_with_gai" class="text-sm">
				{@html overview}
				</p>


				<p class="text-xs font-bold mt-12">
					Planet Details
				</p>

				<p id="details_desc_with_gai" class="text-sm">
				{@html details}
				</p>
			</div>
		</div>
	</div>
{/if}