<script>
	import { onMount } from 'svelte';
	import '../app.css';

	onMount(() => {
		let theme = localStorage.getItem('theme');
		if (!theme && window.matchMedia('(prefers-color-scheme: dark)').matches) {
			document.querySelector('html')?.setAttribute('data-theme', 'dark');
		} else {
			if (theme === 'dark') {
				document.querySelector('html')?.setAttribute('data-theme', theme);
			}
		}



	})
</script>


<slot></slot>
