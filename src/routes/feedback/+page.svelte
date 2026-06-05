<script>
	import { PUBLIC_API_URL } from '$env/static/public';

	let user = null;

	if (typeof localStorage !== 'undefined') {
		user = JSON.parse(localStorage.getItem('user'));
	}

	const modulos = [
		'Dashboard',
		'Compras',
		'Stock',
		'Remitos',
		'Mantenimiento',
		'Formularios',
		'Otro'
	];

	const categorias = [
		'Error',
		'Duda',
		'Sugerencia',
		'Nueva Funcionalidad'
	];

	let feedback = {
		modulo: '',
		categoria: 'Sugerencia',
		descripcion: ''
	};

	let guardando = false;

	async function enviarFeedback() {

		if (!feedback.modulo || !feedback.descripcion.trim()) {

			alert('Complete todos los campos');
			return;
		}

		try {

			guardando = true;

			const payload = {
				tipo: 'feedback',
				fecha: new Date().toISOString(),
				usuario: user?.usuario || 'Desconocido',
				modulo: feedback.modulo,
				categoria: feedback.categoria,
				descripcion: feedback.descripcion,
				estado: 'Pendiente',
				version: 'v1.0',
				pagina: window.location.pathname
			};

			const response = await fetch(
				PUBLIC_API_URL,
				{
					method: 'POST',
					body: JSON.stringify(payload)
				}
			);

			const result = await response.json();

			alert(result.message);

			feedback = {
				modulo: '',
				categoria: 'Sugerencia',
				descripcion: ''
			};

		} catch (error) {

			console.error(error);

			alert('Error al enviar feedback');

		} finally {

			guardando = false;
		}
	}
</script>

<div class="container">

	<h1>Centro de Feedback ERP</h1>

	<p class="subtitulo">
		Reporte errores, dudas, sugerencias o nuevas funcionalidades.
	</p>

	<div class="card">

		<select bind:value={feedback.modulo}>
			<option value="">Seleccione módulo</option>

			{#each modulos as modulo}
				<option value={modulo}>
					{modulo}
				</option>
			{/each}
		</select>

		<select bind:value={feedback.categoria}>
			{#each categorias as categoria}
				<option value={categoria}>
					{categoria}
				</option>
			{/each}
		</select>

		<textarea
			rows="6"
			placeholder="Describa el problema, duda o mejora..."
			bind:value={feedback.descripcion}
		></textarea>

		<button
			class="guardar"
			on:click={enviarFeedback}
			disabled={guardando}
		>
			{guardando
				? 'Enviando...'
				: 'Enviar Feedback'}
		</button>

	</div>

</div>

<style>

	.container {
		max-width: 900px;
		margin: 0 auto;
		padding: 20px;
	}

	h1 {
		margin-bottom: 6px;
		color: #111827;
	}

	.subtitulo {
		color: #6b7280;
		margin-bottom: 24px;
	}

	.card {
		background: white;
		padding: 24px;
		border-radius: 14px;
		box-shadow: 0 4px 12px rgba(0,0,0,.08);

		display: flex;
		flex-direction: column;
		gap: 14px;
	}

	select,
	textarea {
		padding: 12px;
		border: 1px solid #d1d5db;
		border-radius: 10px;
		font-size: 14px;
	}

	textarea {
		resize: vertical;
	}

	.guardar {
		background: #2563eb;
		color: white;
		border: none;
		padding: 12px;
		border-radius: 10px;
		font-weight: 600;
		cursor: pointer;
	}

	.guardar:hover {
		background: #1d4ed8;
	}

	.guardar:disabled {
		opacity: .7;
		cursor: not-allowed;
	}
</style>