<script>
	import '../app.scss';
	import Nav from '$lib/Nav.svelte';
</script>

<div class="container">
	<Nav>
		<slot /></Nav
	>
</div>

<slot />

<style>
	.container {
		height: 100vh;
		background: #2c5d63;
	}
</style>
