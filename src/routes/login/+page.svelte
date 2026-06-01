<script>
	import { usuario as usuarioStore } from '$lib/stores/auth';
	import { goto } from '$app/navigation';

	let usuario = '';
	let password = '';
	let error = '';
	let loading = false;

	import { PUBLIC_API_URL } from '$env/static/public';

	async function iniciarSesion() {

		error = '';
		loading = true;

		try {

			const response = await fetch(PUBLIC_API_URL, {
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

			if (result.debeCambiar === 'SI') {

				goto('/cambiar-password');

			} else {

				goto('/');
}

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
		<div class="logo-mineria">
			⚒️
		</div>
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
		box-sizing: border-box;

		background-image:
			linear-gradient(
				rgba(0,0,0,0.55),
				rgba(0,0,0,0.55)
			),
			url('https://cdn4.mineriaydesarrollo.com/s4/2025/12/26/mineriaydesarrollo/images/22/92/229273_6f7ce29082b616db8fc8b595beed865bf84354c6e3acc80bfccf91cac92105a2/md.webp');

		background-size: cover;
		background-position: center;
		background-repeat: no-repeat;
	}

	.login-card {
		background: rgba(255,255,255,0.95);
		backdrop-filter: blur(10px);

		padding: 40px;
		border-radius: 16px;

		box-shadow:
			0 10px 30px rgba(0,0,0,0.25);

		width: 100%;
		max-width: 380px;

		font-family: Arial, sans-serif;
	}

	h1 {
		margin-top: 0;
		margin-bottom: 10px;
		color: #111827;
		font-size: 32px;
		font-weight: 700;
		text-align: center;
	}

	p {
		color: #6b7280;
		margin-bottom: 24px;
		text-align: center;
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

	.logo-mineria {
		font-size: 42px;
		text-align: center;
		margin-bottom: 12px;
	}
</style>