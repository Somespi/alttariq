<script lang="ts">
	$: select = 'Unselected';
	export let selected: string;
	$: selected = select;
	export let options: string[];
</script>

<div class="dropdown w-full">
	<input
		class="input input-primary input-disabled w-full cursor-pointer text-base-content"
		contenteditable="false"
		tabindex="0"
		bind:value={select}
	/>
	<!-- svelte-ignore a11y-no-noninteractive-tabindex -->
	<ul
		tabindex="0"
		class="menu dropdown-content z-[1] w-full rounded-box bg-base-100 p-2 shadow"
	>
		{#each options as option}
			<li>
				<!-- svelte-ignore a11y-missing-attribute -->
				<!-- svelte-ignore a11y-click-events-have-key-events -->
				<!-- svelte-ignore a11y-no-static-element-interactions -->
				<a on:click={() => (select = option)}>
					{option}
				</a>
			</li>
		{/each}
		<!-- svelte-ignore a11y-missing-attribute -->
		<!-- svelte-ignore a11y-missing-attribute -->
	</ul>
</div>
