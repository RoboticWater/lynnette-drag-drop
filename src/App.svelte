<script>
	import LiveUpdate from './LiveUpdate.svelte'
	import RequestAndSend from './RequestAndSend.svelte'
	import DragDrop from './DragDrop.svelte'

	let tab = 1;
</script>

<div class="root">
	<div class="tabs">
		<button on:click={() => tab = 0}>Collab Live</button>
		<button on:click={() => tab = 1}>Collab Request</button>
		<button on:click={() => tab = 2}>Normal</button>
	</div>
	{#if tab === 0}
		<LiveUpdate/>
	{:else if tab === 1}
		<RequestAndSend/>
	{:else}
		<DragDrop/>
	{/if}
</div>

<style>
	.root {
		width: 100%;
		height: 100%;
	}
	.tabs {
		display: flex;
		position: fixed;
		bottom: 15px;
		left: 20px;
		font-size: 20px;
		z-index: 1000;
	}
</style>