<script lang="ts">
	import { goto } from '$app/navigation';
    import DropDownSelect from './DropDownSelect.svelte';

	$: selectedGrade = 'Unselected';
	let grades: string[] = ['Elementary', 'Middle-School', 'High-School', 'College'];

    function transitionToNextPage() {
        window.sessionStorage.setItem('grade', selectedGrade.toLowerCase());
        goto('/explore');
    }
</script>

<div class="flex flex-col items-center justify-center content-center h-screen w-screen">
	<div class="flex flex-col gap-y-16">
		<DropDownSelect bind:selected={selectedGrade} options={grades}/>
        <button class="btn btn-primary" disabled={selectedGrade == "Unselected"} on:click={transitionToNextPage}>
            Explore the Sky
        </button>
	</div>

</div>
