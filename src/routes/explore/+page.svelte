<script lang="ts">
	import SkyView from '$lib/SkyView.svelte';
	import { onMount } from 'svelte';
	import Icon from '@iconify/svelte';
	import { goto } from '$app/navigation';
	import * as THREE from 'three';
	let scene: any;
	import Card from '$lib/Card.svelte';
	let camera: any;
	//@ts-ignore
	import * as planetsJSON from '$lib/planets.json';
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
	$: is_comparing = false;

	const generateNasaDataIntoString = (data: {}) => {
		let string = '';
		for (let key in data) {
			// @ts-ignore
			string += key + ': ' + data[key] + '\n';
		}
		return string;
	};

	onMount(() => {
		if (sessionStorage.getItem('grade') == null) {
			goto('/');
		}
		let url = new URL(window.location.href);
		let planet_id = url.searchParams.get('id');
		if (planet_id) id = parseInt(planet_id);

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
		fetch('https://long-clari-alfarsi-04de02c2.koyeb.app' + '/prompt', {
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
					', Use the data provided by NASA to authoritise your work, the data is: ' +
					generateNasaDataIntoString(planets[id].nasa_data),
				command:
					'You are going to act as only-education tailor, your primary goal is to generate an overview description of the given planet, for example: \
					if you were asked to generate an overview for the planet : Kepler-22 b, then you will respond with something like: A potentially rocky world, larger than Earth and Super Earth, A possible ocean world orbiting in the habitable zone—the region around a star where the temperature is right for liquid water, a requirement for life on Earth\
					.\nHighlighting the given planet in the overview: Kepler-22 b with its type and size, and its key features is embedded between <b> and </b> to highlight it, also, make sure to not exceed the 200 words limit, try to use emojis if it fits for the educational grade.\
					Also, make sure to best-fit the content with the given educational grade, , try to use emojis if it fits for the educational grade.',
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
	};

	const detailsGeneration = (id: number) => {
		fetch('https://long-clari-alfarsi-04de02c2.koyeb.app' + '/prompt', {
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
					', Use the data provided by NASA to authoritise your work, the data is: ' +
					generateNasaDataIntoString(planets[id].nasa_data),
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
	};

	const tidyUpDetails = () => {
		fetch('https://long-clari-alfarsi-04de02c2.koyeb.app' + '/prompt', {
			method: 'POST',
			headers: {
				'Content-Type': 'application/json'
			},
			body: JSON.stringify({
				prompt:
					'Tidy up the given details: ' +
					displayed_details +
					". Don't forget to best-fit the content with " +
					sessionStorage.getItem('grade') +
					' educational grade, ' +
					', Use the data provided by NASA to authoritise your work, the data is: ' +
					generateNasaDataIntoString(planets[id].nasa_data),
				command:
					'You are going to act as an AI tool to extract the key words and featured properties from the given details, for example, you are going to generate a list of the properties with minimal explination for them, make sure to best-fit the content with the given educational grade and it doesnt exceed 100 words.',
				message:
					"Sure, I'm going to act and behave as described and generate the list of the properties to best-fit the given educational grade and make sure its less than 100 words."
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
	};

	const promptGeneration = (id: number) => {
		fetch('https://long-clari-alfarsi-04de02c2.koyeb.app' + '/prompt', {
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
					' educational grade, the question is: ' +
					promptG +
					', Use the data provided by NASA to authoritise your work, the data is: ' +
					generateNasaDataIntoString(planets[id].nasa_data),
				command:
					'You are going to act as 1-conversation chatbot educational chatbot, your primary goal is to answer the given question and make sure to best-fit the content with the given educational grade, with minimal explaination, try to answer with 50 words for the given prompt, also, use the data provided by NASA to authoritise your work.',
				message:
					"Ok, I'm going to act and behave as described and generate the answer to best-fit the given educational grade and make sure its less than 50 words."
			})
		})
			.then((res) => res.json())
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
			});
	};

	const onPlanetClick = (planet_id: number) => {
		let url = new URL(window.location.href);
		url.searchParams.set('id', planet_id.toString());
		window.history.pushState(null, '', url);
		id = planet_id;

		overview = '';
		displayed_overview = '';
		details = '';
		displayed_details = '';
		displayed_tidy = '';
		displayed_prompt = '';
		promptG = '';

		overviewGeneration(id);
		detailsGeneration(id);
		
	};

	const backToDiscovery = () => {
		let url = new URL(window.location.href);
		url.searchParams.delete('id');
		window.history.pushState(null, '', url);
		id = -1;
	};
	function estimateGravity(mass: string, radius: string) {
		const earthGravity = 9.8;

		const massValue = parseFloat(mass.slice(1));
		console.log(mass, radius);
		if (radius == '---' || mass == '---') {
			return 'Unknown';
		}

		const [radiusValue, radiusUncertainty] = radius.split('±').map(parseFloat);

		const radiusLowerBound = radiusValue - radiusUncertainty;
		const radiusUpperBound = radiusValue + radiusUncertainty;

		const surfaceGravityLower = earthGravity * (massValue / Math.pow(radiusUpperBound, 2));
		const surfaceGravityUpper = earthGravity * (massValue / Math.pow(radiusLowerBound, 2));
		return '~ ' + surfaceGravityLower.toFixed(2).toString() + ' g';
	}

	function compareToEarth(id: number) {
		if (planets[id].nasa_data.radius_earth_compared == '---') {
			alert(
				"NASA does not currently have data on the radius of the planet compared to Earth."
			)
		} else {
			is_comparing = !is_comparing;
		}
	}

	function luanchSimulation() {
		window.open("https://vqr-87.github.io/At-tariq-simulation/?type=" + planets[id].type + "&gravity=" + planets[id].nasa_data.gravity, "_blank");
	}
</script>

{#if !is_loaded}
	Loading...
{:else if id == -1}
	
<SkyView {scene} {camera} {planets} {onPlanetClick} />
	<button class="btn btn-outline btn-primary btn-sm absolute w-auto p-2 top-5 left-5" on:click={() => goto('/')}>
		Back to Homepage
	</button>
{:else}

	<div class="flex h-screen w-full flex-row">
		<div class="absolute m-5 w-auto p-2">
			<button class="btn btn-outline btn-primary btn-sm" on:click={backToDiscovery}>
				Back to Discovering
			</button>

			<button class="btn btn-outline btn-accent btn-sm" on:click={luanchSimulation}>
				Launch Simulation
			</button>

			

			<button class="btn {is_comparing ? '' : 'btn-outline'} btn-secondary btn-sm" on:click={() => compareToEarth(id)}>
				{#if !is_comparing}
					Compare to Earth
				{:else}
					Close Comparison 
				{/if}
			</button>
		</div>
		<div class=" h-full w-[63%] bg-slate-950">
	
			<DecidePlanet {is_comparing} {THREE} {scene} {camera} planet={planets[id]} />
		</div>

		<div class="h-full flex-1 p-3" style="display: {is_comparing ? 'none' : 'block'};">
			<div class="h-full w-full overflow-y-auto rounded-2xl bg-base-200 p-3 text-base-content">
				<h1 class="text-2xl font-medium">
					{planets[id].name}
				</h1>

				<p class="mt-12 text-xs font-bold">Planet Overview</p>
				<p id="small_desc_with_gai" class="text-sm leading-relaxed">
					{#if overview == ''}
						Generating...
					{:else}
						{@html highlight(displayed_overview)}
					{/if}
				</p>

				<div class="2-full mt-12 grid h-64 grid-cols-2 grid-rows-2 gap-5">
					<Card
						name="GRAVITY"
						content={estimateGravity(
							planets[id].nasa_data.mass_earth_compared,
							planets[id].nasa_data.radius_earth_compared
						)}
					>
						<Icon icon="material-symbols:specific-gravity" />
					</Card>

					<Card
						name="TEMP."
						content={planets[id].nasa_data.temperature == '---'
							? 'N/A'
							: planets[id].nasa_data.temperature}
					>
						<Icon icon="fluent:temperature-32-filled" />
					</Card>

					<Card
						name="ORBIT PERIOD"
						content={planets[id].nasa_data.orbit_period.includes('±')
							? planets[id].nasa_data.orbit_period.split('±')[0] + ' days'
							: planets[id].nasa_data.orbit_period.split('+')[0] + ' days'}
					>
						<Icon icon="mdi:orbit" />
					</Card>

					<Card name="EARTH MASS" content="{planets[id].nasa_data.mass_earth_compared} times">
						<Icon icon="material-symbols:globe-uk-sharp" />
					</Card>
				</div>

				<p
					class="mt-12 flex snap-center flex-row content-center items-center space-x-4 self-center text-xs font-bold"
				>
					<span>Planet Details</span>

					<button
						class="btn btn-square btn-primary btn-xs text-purple-600 {displayed_details !=
							details && displayed_details != ''
							? 'btn-disabled'
							: ''}"
						on:click={tidyUpDetails}
						disabled={displayed_details != details && displayed_details != ''}
					>
						<Icon icon="flowbite:wand-magic-sparkles-solid" />
					</button>
				</p>

				<p id="details_desc_with_gai" class="text-sm leading-relaxed">
					{#if details == '' && displayed_tidy == ''}
						Generating...
					{:else if displayed_tidy == ''}
						{@html highlight(displayed_details)}
					{:else}
						{@html highlight(displayed_tidy)}
					{/if}
				</p>

				<div class="2-full mt-12 flex h-64 flex-col rounded-lg bg-base-300 p-3">
					<h1 class="flex content-center items-center gap-x-2 self-center text-lg font-medium">
						<Icon icon="bxs:planet" />
						Planet AI
					</h1>
					<div class="join mt-5">
						<input
							bind:value={promptG}
							class="input join-item input-bordered w-full"
							placeholder="Ask a question..."
						/>
						<button on:click={() => promptGeneration(id)} class="btn join-item rounded-r-lg"
							>Send</button
						>
					</div>
					<div
						class="mt-3 flex h-32 w-full flex-row overflow-y-auto rounded-lg bg-purple-50 bg-opacity-40 p-3"
					>
						<div class="h-full text-violet-300">
							<span class="animate-pulse">
								<Icon icon="mingcute:sparkles-2-fill" />
							</span>
							<div class="mb-2 flex-1 overflow-y-auto text-black">
								{@html highlight(displayed_prompt)}
							</div>
						</div>
					</div>
				</div>
			</div>
		</div>
	</div>
{/if}
