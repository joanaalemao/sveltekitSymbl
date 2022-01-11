<script>
	import Nav from '$lib/Nav.svelte';
</script>

<div class="container">
	<Nav />
	
	<img src="bw1.png" alt="" />
	<slot />
</div>

<style>
	.container {
		background: grey;
		/* background-image:image(../static); */
		font-family: Arial, Helvetica, sans-serif;
		text-align: center;
	}

	img {
		height: 50%;
	}
</style>
