<script>
	// props
	export let extraClass = '';
</script>

<div class={extraClass}>
	<slot />
</div>

<style>
	.start-ctl {
		display: flex;
		justify-content: center;
		align-items: center;
		margin: 4rem 0;
	}
</style>
