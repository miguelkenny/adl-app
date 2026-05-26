<script>
	import { goto } from '$app/navigation';

	let password1 = '';
	let password2 = '';

	let error = '';
	let success = '';
	let loading = false;

	import { PUBLIC_API_URL } from '$env/static/public';

	const user =
		JSON.parse(localStorage.getItem('user'));

	async function cambiarPassword() {

		error = '';
		success = '';

		if (!password1 || !password2) {

			error = 'Completar todos los campos';
			return;
		}

		if (password1 !== password2) {

			error = 'Las contraseñas no coinciden';
			return;
		}

		if (password1.length < 4) {

			error = 'Mínimo 4 caracteres';
			return;
		}

		loading = true;

		try {

			const response = await fetch(PUBLIC_API_URL, {
				method: 'POST',
				body: JSON.stringify({
					action: 'cambiarPassword',
					usuario: user.usuario,
					password: password1
				})
			});

			const result = await response.json();

			if (result.success) {

				user.debeCambiar = 'NO';

				localStorage.setItem(
					'user',
					JSON.stringify(user)
				);

				success = 'Contraseña actualizada';

				setTimeout(() => {

					goto('/');

				}, 1200);

			} else {

				error = result.message || 'Error';
			}

		} catch (err) {

			error = 'Error de conexión';
		}

		loading = false;
	}
</script>

<div class="container">

	<div class="card">

		<h1>Cambiar contraseña</h1>

		<p>
			Debes crear una contraseña personal.
		</p>

		<input
			type="password"
			placeholder="Nueva contraseña"
			bind:value={password1}
		/>

		<input
			type="password"
			placeholder="Repetir contraseña"
			bind:value={password2}
		/>

		<button
			on:click={cambiarPassword}
			disabled={loading}
		>
			{#if loading}
				Guardando...
			{:else}
				Guardar contraseña
			{/if}
		</button>

		{#if error}
			<div class="error">
				{error}
			</div>
		{/if}

		{#if success}
			<div class="success">
				{success}
			</div>
		{/if}

	</div>

</div>

<style>

	.container {
		min-height: 100vh;
		display: flex;
		justify-content: center;
		align-items: center;
		background: #f3f4f6;
		padding: 20px;
	}

	.card {
		background: white;
		padding: 40px;
		border-radius: 16px;
		width: 100%;
		max-width: 420px;
		box-shadow: 0 10px 30px rgba(0,0,0,0.1);
		font-family: Arial, sans-serif;
	}

	h1 {
		margin-top: 0;
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
	}

	.error {
		margin-top: 16px;
		background: #fee2e2;
		color: #991b1b;
		padding: 10px;
		border-radius: 8px;
	}

	.success {
		margin-top: 16px;
		background: #dcfce7;
		color: #166534;
		padding: 10px;
		border-radius: 8px;
	}

</style>