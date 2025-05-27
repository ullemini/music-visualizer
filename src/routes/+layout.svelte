<script>
	import '../app.css';
	import { base } from '$app/paths';
	let { children } = $props();
</script>

<header>
<nav>
	<a href="{base}/aoudio-test">aoudio-test</a>
	<a href="{base}/">back</a>
</nav>
</header>

{@render children()}

<style>
	nav{
		z-index: 1;
		display: block;
		color: aqua;
		text-align: center;
	}
</style>