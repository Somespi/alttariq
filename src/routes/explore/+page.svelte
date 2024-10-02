<script lang="ts">
	import SkyView from '$lib/SkyView.svelte';
	import { onMount } from 'svelte';
	import Icon from '@iconify/svelte';
	import { goto } from '$app/navigation';
	import Planet from '$lib/Planet.svelte';
	import PlanetFromModel from '$lib/PlanetFromModel.svelte';
	//@ts-ignore
	import * as planetsJSON from "$lib/planets.json"; 
	import DecidePlanet from '$lib/DecidePlanet.svelte';
	//@ts-ignore
	let planets = planetsJSON['default'];
	$: id = -1;
	$: is_loaded = false;
	$: overview = '';
	$: displayed_overview = '';
	$: details = '';
	$: displayed_details = '';
	$: displayed_tidy = '';
	$: displayed_prompt = '';
	$: promptG = '';

	const generateNasaDataIntoString = (data: {}) => {
		let string = '';
		for (let key in data) {
			// @ts-ignore
			string += key + ': ' + data[key] + '\n';
		}
		return string;
	}
	
	onMount(() => {
		if (sessionStorage.getItem('grade') == null) {
			goto('/');
		}
		let url = new URL(window.location.href);
		let planet_id = url.searchParams.get('id');
		if (planet_id) {
			id = parseInt(planet_id);
		}
		is_loaded = true;
		if (is_loaded && id != -1) {
			onPlanetClick(id);
		}
	});

	const highlight = (string: string) => {
		return string.replace(/\*\*(.*?)\*\*/g, '<b>$1</b>');
	};

	const typewriteString = (string: string) => {
		let i = 0;
		let timer = setInterval(() => {
			if (i < string.length) {
				document.getElementById('small_desc_with_gai')!.innerHTML += string.charAt(i);
				i++;
			} else {
				clearInterval(timer);
			}
		}, 10);
	};

	const overviewGeneration = (id: number) => {
		fetch('http://localhost:8080' + '/prompt', {
			method: 'POST',
			headers: {
				'Content-Type': 'application/json'
			},
			body: JSON.stringify({
				prompt:
					'Generate an overview for the given planet: ' +
					planets[id].name +
					', and make sure to best-fit the content with the: ' +
					sessionStorage.getItem('grade') +
					' educational grade,' + 
					", Use the data provided by NASA to authoritise your work, the data is: " + generateNasaDataIntoString(planets[id].nasa_data),
				command:
					'You are going to act as only-education tailor, your primary goal is to generate an overview description of the given planet, for example: \
					if you were asked to generate an overview for the planet : Kepler-22 b, then you will respond with something like: A potentially rocky world, larger than Earth and Super Earth, A possible ocean world orbiting in the habitable zone—the region around a star where the temperature is right for liquid water, a requirement for life on Earth\
					.\nHighlighting the given planet in the overview: Kepler-22 b with its type and size, and its key features is embedded between <b> and </b> to highlight it, also, make sure to not exceed the 200 words limit.\
					Also, make sure to best-fit the content with the given educational grade.',
				message:
					"Ok, I'm going to act and behave as described and generate the overview to best-fit the given educational grade."
			})
		})
			.then((res) => res.json())
			.then((data) => {
				overview = data['ai_model'];
				console.log(data['ai_model']);
				console.log(data);
				let i = 0;
				const timer = setInterval(() => {
					if (i < overview.length) {
						displayed_overview += overview.charAt(i);
						i++;
					} else {
						clearInterval(timer);
					}
				}, 10);
			});
	}

	const detailsGeneration = (id: number) => {
		fetch('http://localhost:8080' + '/prompt', {
			method: 'POST',
			headers: {
				'Content-Type': 'application/json'
			},
			body: JSON.stringify({
				prompt:
					'Generate a detailed description for the given planet: ' +
					planets[id].name +
					', and make sure to best-fit the content with the: ' +
					sessionStorage.getItem('grade') +
					' educational grade,' + 
					", Use the data provided by NASA to authoritise your work, the data is: " + generateNasaDataIntoString(planets[id].nasa_data),
				command:
					"You are going to act as only-education tailor, your primary goal is to generate a detailed description of the given planet, for example: \
					if you were asked to generate details that always includes (type, size, gravity, mass, orbit, distance, temperature, atmosphere, discovery, further research) and more for the planet : Kepler-22 b, then you will respond with something that includes its properties such as gravity, mass and other primary things, don't forget to mention the numbers and use the information that i am going to give you which are from NASA to authoritise your work. \
					.\nHighlighting the given planet properties in the details with its type and size, and its key features is embedded between <b> and </b> to highlight it. (use html)\
					Also, make sure to best-fit the content with the given educational grade.",
				message:
					"Ok, I'm going to act and behave as described and generate the overview to best-fit the given educational grade with using html for formatting the highlighting, and also, i will also be using NASA's data to authoritise my work and I will always include data about type, size, gravity, mass, orbit, distance, temperature, atmosphere, discovery, further research and more."
			})
		})
			.then((res) => res.json())
			.then((data) => {
				details = data['ai_model'];
				console.log(data['ai_model']);
				console.log(data);
				let i = 0;
				const timer = setInterval(() => {
					if (i < details.length) {
						displayed_details += details.charAt(i);
						i++;
					} else {
						clearInterval(timer);
					}
				}, 10);
			});
	}

	const tidyUpDetails = () => {
		fetch('http://localhost:8080' + '/prompt', {
			method: 'POST',
			headers: {
				'Content-Type': 'application/json'
			},
			body: JSON.stringify({
				prompt: 'Tidy up the given details: ' + displayed_details + ". Don't forget to best-fit the content with "+ sessionStorage.getItem('grade') +" educational grade, " +", Use the data provided by NASA to authoritise your work, the data is: " + generateNasaDataIntoString(planets[id].nasa_data),
				command: "You are going to act as an AI tool to extract the key words and featured properties from the given details, for example, you are going to generate a list of the properties with minimal explination for them, make sure to best-fit the content with the given educational grade and it doesnt exceed 100 words.",
				message: "Sure, I'm going to act and behave as described and generate the list of the properties to best-fit the given educational grade and make sure its less than 100 words."
			})
		})
			.then((res) => res.json())
			.then((data) => {
				let i = 0;
				const timer = setInterval(() => {
					if (i < data['ai_model'].length) {
						displayed_tidy += data['ai_model'].charAt(i);
						i++;
					} else {
						clearInterval(timer);
					}
				}, 10);
			});
	
	}

	const promptGeneration = (id: number) => {
		fetch('http://localhost:8080' + '/prompt', {
			method: 'POST',
			headers: {
				'Content-Type': 'application/json'
			},
			body: JSON.stringify({
				prompt:
					'Answer the following question for the given planet: ' +
					planets[id].name +
					', and make sure to best-fit the content with the: ' +
					sessionStorage.getItem('grade') +
					' educational grade, the question is: ' + promptG + 
					", Use the data provided by NASA to authoritise your work, the data is: " + generateNasaDataIntoString(planets[id].nasa_data),
				command: "You are going to act as 1-conversation chatbot educational chatbot, your primary goal is to answer the given question and make sure to best-fit the content with the given educational grade, with minimal explaination, try to answer with 50 words for the given prompt, also, use the data provided by NASA to authoritise your work.",
				message: "Ok, I'm going to act and behave as described and generate the answer to best-fit the given educational grade and make sure its less than 50 words."
			})
		}).then((res) => res.json())
		.then((data) => {
			displayed_prompt = '';
			let i = 0;
				const timer = setInterval(() => {
					if (i < data['ai_model'].length) {
						displayed_prompt += data['ai_model'].charAt(i);
						i++;
					} else {
						clearInterval(timer);
					}
				}, 15);
		})
	}
	const onPlanetClick = (planet_id: number) => {
		let url = new URL(window.location.href);
		url.searchParams.set('id', planet_id.toString());
		window.history.replaceState(null, '', url);
		id = planet_id;

		overviewGeneration(id);
		detailsGeneration(id);
	};
</script>

{#if !is_loaded}
	Loading...
{:else if id == -1}
	<SkyView {planets} {onPlanetClick} />
{:else}
	<div class="flex h-screen w-full flex-row">
		<div class=" h-full w-[63%] bg-slate-950">
			<DecidePlanet planet={planets[id]} /> 
		</div>

		<div class="h-full flex-1 p-3">
			<div class="h-full w-full overflow-y-auto rounded-2xl bg-base-200 p-3 text-accent-content">
				<h1 class="text-2xl font-medium">
					{planets[id].name}
				</h1>

				<p class="mt-12 text-xs font-bold">Planet Overview</p>
				<p id="small_desc_with_gai" class="text-sm leading-relaxed">
					{@html highlight(displayed_overview)}
				</p>

				<p class="mt-12 text-xs font-bold flex flex-row space-x-4 content-center snap-center self-center items-center">
					<span>Planet Details</span>
				
						<button class="btn btn-xs btn-primary text-purple-600 btn-square {displayed_details != details && displayed_details != '' ? 'btn-disabled' : ''}" on:click={tidyUpDetails} disabled={displayed_details != details && displayed_details != ''}>
							<Icon icon="flowbite:wand-magic-sparkles-solid" />
						</button>
				

				</p>

				<p id="details_desc_with_gai" class="text-sm leading-relaxed">
					{#if displayed_tidy == ''}
						{@html highlight(displayed_details)}
					{:else}
						{@html highlight(displayed_tidy)}
					{/if}
				</p>


				<div class="mt-12 h-64 2-full rounded-lg bg-base-300 p-3 flex flex-col">
					<h1 class="text-lg font-medium flex content-center self-center items-center gap-x-2">
						<Icon icon="bxs:planet" />
						Planet AI 
					</h1>
					<div class="join mt-5">
						<input bind:value={promptG} class="input input-bordered join-item w-full" placeholder="Ask a question..." />
						<button on:click={() => promptGeneration(id)} class="btn join-item rounded-r-lg">Send</button>
					  </div>
					<div class="flex flex-row w-full mt-3 bg-purple-50 p-3 bg-opacity-40 h-32 rounded-lg overflow-y-auto">
						<div class="h-full text-violet-300" >
							<span class="animate-pulse">
								<Icon icon="mingcute:sparkles-2-fill" />
							</span>
							<div class="flex-1 text-black overflow-y-auto mb-2">
								{@html highlight(displayed_prompt)}
							</div>
						</div>
					</div>
				</div>

				


				
			</div>
		</div>
	</div>
{/if}
