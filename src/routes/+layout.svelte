<script lang="ts">
	import "../main.css";
	import { resolve } from "$app/paths";
	import favicon from "$lib/assets/favicon.svg";

	let { children } = $props();
</script>

<svelte:head>
	<link rel="icon" href={favicon} />
</svelte:head>
<nav>
	<ul>
		<li>
			<a href={resolve("/")}>home</a>
		</li>
		<li>
			<a href={resolve("/images")}>Images</a>
		</li>
	</ul>
</nav>
{@render children()}

<style lang="scss">
	nav {
		width: 100%;
		ul {
			display: flex;
			flex-direction: row;
			width: 100%;
			justify-content: center;
			align-items: center;
			height: 60px;
		}
		li {
			min-width: 200px;
			text-align: center;
		}
	}
</style>
