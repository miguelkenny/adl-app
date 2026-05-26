<script>
	import Header from './Header.svelte';
	import './layout.css';

	import { onMount } from 'svelte';
	import { goto } from '$app/navigation';
	import { page } from '$app/state';

	let { children } = $props();

	let cargando = $state(true);

	onMount(() => {

		const user = localStorage.getItem('user');

		// si estamos en login
		if (window.location.pathname === '/login') {

			cargando = false;
			return;
		}

		// si no hay sesión
		if (!user) {

			goto('/login');
			return;
		}

		cargando = false;
	});
</script>

{#if cargando}

	<div class="loading">
		Cargando...
	</div>

{:else}

	<div class="app">

		{#if page.url.pathname !== '/login'}
			<Header />
		{/if}

		<main>
			{@render children()}
		</main>

		<footer></footer>

	</div>

{/if}

<style>

	.loading {
		height: 100vh;
		display: flex;
		justify-content: center;
		align-items: center;
		font-family: Arial, sans-serif;
		font-size: 18px;
	}

	.app {
		display: flex;
		flex-direction: column;
		min-height: 100vh;
	}

	main {
		flex: 1;
		display: flex;
		flex-direction: column;
		width: 100%;
		box-sizing: border-box;
	}

	footer {
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: center;
		padding: 12px;
	}

	@media (min-width: 480px) {

		footer {
			padding: 12px 0;
		}
	}

</style>