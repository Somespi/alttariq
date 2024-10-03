<script lang="ts">
	export let name;
	export let content;
</script>

<div class="flex flex-col rounded-lg bg-base-300 text-base-content">
	<p class="flex flex-row content-center items-center gap-x-2 p-2 text-xs">
		<slot />
		<span class="card-tile font-bold">
			{name}
		</span>
	</p>
	<div
		class="flex flex-1 content-center items-center justify-center font-mono font-semibold text-primary"
	>
		{content}
	</div>
</div>
