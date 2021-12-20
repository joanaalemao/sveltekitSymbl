<script>
	import Nav from '$lib/Nav.svelte';
</script>

<div class="container">

<slot />
<Nav />
</div>

<style>
    .container{
        height: 100vh;
        background: #23595B; 
    }

</style>
