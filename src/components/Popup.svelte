<script>
    export let visible = false;
    export let data = {};
</script>

{#if visible}
<div class="popup">
    <button on:click={() => visible = false}>Close</button>
    {#each Object.entries(data) as [type, count]}
    <p>{type}: {count}</p>
    {/each}
</div>
{/if}

<style>
.popup {
    position: fixed;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    background-color: white;
    border: 1px solid black;
    padding: 20px;
    z-index: 100;
}
</style>
