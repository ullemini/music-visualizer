<script>
	import '../app.css';
	import { base } from '$app/paths';
	let { children } = $props();
</script>

<nav>
	<a href="{base}/aoudio-test">aoudio-test</a>
	<a href="{base}/">back</a>

</nav>

{@render children()}

<style>
	nav{
		color: aqua;
		text-align: center;
	}
</style>