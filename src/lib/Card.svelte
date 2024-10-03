<script lang="ts">
    export let name; 
    export let content; 
</script>

<div class="rounded-lg bg-base-300 text-base-content flex flex-col">
    <p class="flex flex-row gap-x-2 content-center p-2 items-center text-xs">
        <slot />
        <span class="font-bold card-tile">
            {name}
        </span>
    </p>
    <div class="flex-1 flex content-center justify-center items-center text-primary font-mono font-semibold">
        {content}
    </div>
</div>