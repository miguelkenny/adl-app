<script>
	import { usuario as usuarioStore } from '$lib/stores/auth';
	import { goto } from '$app/navigation';

	let usuario = '';
	let password = '';
	let error = '';
	let loading = false;

	const API_URL =
		'https://script.google.com/macros/s/AKfycbyOL2jeaK79ebykD_zfx1Eqg1yFoBFvZQPoqnL4MAlG--CPtPiBg93J6AcCpRTFbw6WbQ/exec';

	async function iniciarSesion() {

		error = '';
		loading = true;

		try {

			const response = await fetch(API_URL, {
				method: 'POST',
				headers: {
					'Content-Type': 'text/plain;charset=utf-8'
				},
				body: JSON.stringify({
					action: 'login',
					usuario,
					password
				})
			});

			const result = await response.json();

			if (result.success) {

				localStorage.setItem(
				'user',
				JSON.stringify(result)
			);

			usuarioStore.set(result);

			goto('/');

			} else {

				error = result.message;
			}

		} catch (err) {

			error = 'Error de conexión';
		}

		loading = false;
	}
</script>

<div class="login-container">

	<div class="login-card">

		<h1>ADL ERP</h1>

		<p>Iniciar sesión</p>

		<input
			type="text"
			placeholder="Usuario"
			bind:value={usuario}
		/>

		<input
			type="password"
			placeholder="Contraseña"
			bind:value={password}
		/>

		<button
			on:click={iniciarSesion}
			disabled={loading}
		>
			{#if loading}
				Ingresando...
			{:else}
				Ingresar
			{/if}
		</button>

		{#if error}
			<div class="error">
				{error}
			</div>
		{/if}

	</div>

</div>

<style>

	.login-container {
		min-height: 100vh;
		display: flex;
		justify-content: center;
		align-items: center;
		background: #f3f4f6;
		padding: 20px;
		box-sizing: border-box;
	}

	.login-card {
		background: white;
		padding: 40px;
		border-radius: 16px;
		box-shadow: 0 10px 30px rgba(0,0,0,0.1);
		width: 100%;
		max-width: 380px;
		font-family: Arial, sans-serif;
	}

	h1 {
		margin-top: 0;
		margin-bottom: 10px;
		color: #111827;
	}

	p {
		color: #6b7280;
		margin-bottom: 24px;
	}

	input {
		width: 100%;
		padding: 12px;
		margin-bottom: 16px;
		border-radius: 10px;
		border: 1px solid #d1d5db;
		font-size: 14px;
		box-sizing: border-box;
	}

	button {
		width: 100%;
		padding: 12px;
		border: none;
		border-radius: 10px;
		background: #2563eb;
		color: white;
		font-weight: bold;
		cursor: pointer;
		font-size: 15px;
	}

	button:hover {
		opacity: 0.95;
	}

	.error {
		margin-top: 16px;
		background: #fee2e2;
		color: #991b1b;
		padding: 10px;
		border-radius: 8px;
		font-size: 14px;
	}

</style>