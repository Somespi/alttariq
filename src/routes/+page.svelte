<script lang="ts">
	import { goto } from '$app/navigation';
    import DropDownSelect from '../lib/DropDownSelect.svelte';

	$: selectedGrade = 'Unselected';
	let grades: string[] = ['Elementary', 'Middle-School', 'High-School', 'College'];

    function transitionToNextPage() {
        window.sessionStorage.setItem('grade', selectedGrade.toLowerCase());
        goto('/explore');
    }
</script>
<div class="flex flex-col  h-screen w-screen">
    <div class="w-full flex-row top-0 justify-around">
        <h1 class="p-3">At-Tariq Explorer</h1>
        
    </div>
    <div class="flex-1 flex flex-col justify-center items-center content-center">
        <div class="flex flex-col gap-y-16">
            <DropDownSelect bind:selected={selectedGrade} options={grades}/>
            <button class="btn btn-primary" disabled={selectedGrade == "Unselected"} on:click={transitionToNextPage}>
                Explore the Sky
            </button>
        </div>
    </div>

</div>
