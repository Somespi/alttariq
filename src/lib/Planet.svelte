<script>
	export let x;
	export let y;
	export let onClick;
</script>

<!-- svelte-ignore a11y-click-events-have-key-events -->
<!-- svelte-ignore a11y-no-static-element-interactions -->
<div
	on:click={onClick}
    class="absolute w-10 h-10 "
	style="
		position: absolute;
		left: {x}px;
		top: {y}px;
		width: 10px;
		height: 10px;
		border-radius: 50%;
		background-color: yellow; /* Change color as needed */
		box-shadow: 0 0 10px yellow, 0 0 20px yellow; /* Glowing effect */
		cursor: pointer;
	"
></div>
