<script lang="ts">
  import DropDownSelect from './DropDownSelect.svelte';

	$: selectedGrade = 'Unselected';
	let grades: string[] = ['Elementary', 'Middle-School', 'High-School', 'College'];
</script>

<div class="flex flex-col items-center justify-center content-center h-screen w-screen">
	<div class="flex flex-col gap-y-16">
		<DropDownSelect bind:selected={selectedGrade} options={grades}/>
        <button class="btn btn-primary" disabled={selectedGrade == "Unselected"}>
            Explore the Sky
        </button>
	</div>

</div>
