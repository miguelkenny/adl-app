<script>
	import { onMount } from 'svelte';

	let solicitudes = [];
	let loading = true;
	let busqueda = '';
	let solicitudSeleccionada = null;

	const API_URL = 'https://script.google.com/macros/s/AKfycbzqVNelaRN1hL_gDbewXJ9j_L3j64T1c-cYgEeIxUHtjYOgrRsh8VZuFe0MhFCQ4FFjNQ/exec';

	onMount(async () => {

		try {

			const response = await fetch(API_URL);

			const data = await response.json();

			solicitudes = data.map((item) => ({

				...item,

				fechaFormateada: new Date(
					item['Marca temporal']
				).toLocaleDateString('es-AR')

			}))
			.sort(
				(a, b) =>
					new Date(b['Marca temporal']) -
					new Date(a['Marca temporal'])
			);

		} catch (error) {

			console.error(error);

		} finally {

			loading = false;

		}
	});
</script>

<div class="mantenimiento-container">

	<h1>Solicitudes de Mantenimiento</h1>

	<input
		type="text"
		placeholder="Buscar equipo, vehículo o problema..."
		bind:value={busqueda}
	/>

	<p>
		Total visibles:
		{
			solicitudes.filter((item) => {

				const texto = `
					${item['EQUIPMENT / EQUIPO']}
					${item['VEHICLE / VEHICULO']}
					${item['PROBLEM / PROBLEMA / TEMA']}
				`.toLowerCase();

				return texto.includes(
					busqueda.toLowerCase()
				);

			}).length
		}
	</p>

	{#if loading}

		<p>Cargando...</p>

	{:else}

	<div class="table-container">

		<table>

			<thead>

				<tr>
					<th>Fecha</th>
					<th>Equipo</th>
					<th>Vehículo</th>
					<th>Urgencia</th>
					<th>Problema</th>
				</tr>

			</thead>

			<tbody>

				{#each solicitudes.filter((item) => {

					const texto = `
						${item['EQUIPMENT / EQUIPO']}
						${item['VEHICLE / VEHICULO']}
						${item['PROBLEM / PROBLEMA / TEMA']}
					`.toLowerCase();

					return texto.includes(
						busqueda.toLowerCase()
					);

				}) as item}

					<tr
						on:click={() =>
							solicitudSeleccionada = item
						}
					>

						<td>
							{item.fechaFormateada}
						</td>
						<td>
							{item['EQUIPMENT / EQUIPO']}
						</td>

						<td>
							{item['VEHICLE / VEHICULO']}
						</td>

						<td>

							<span
								class={`urgencia ${
									item['URGENCY LEVEL / NIVEL DE URGENCIA']
										?.toLowerCase()
								}`}
							>

								{item['URGENCY LEVEL / NIVEL DE URGENCIA']}

							</span>

						</td>

						<td>

							{
								item['PROBLEM / PROBLEMA / TEMA']
									?.length > 80

									? item['PROBLEM / PROBLEMA / TEMA']
										.slice(0, 80) + '...'

									: item['PROBLEM / PROBLEMA / TEMA']
							}

						</td>

					</tr>

				{/each}

			</tbody>

		</table>

	</div>

	{/if}

</div>

{#if solicitudSeleccionada}

	<div
		class="overlay"
		on:click={() =>
			solicitudSeleccionada = null
		}
	>

		<div
			class="modal"
			on:click|stopPropagation
		>

			<h2>
				Detalle Solicitud
			</h2>

			<p>
				<strong>Fecha:</strong>
				{solicitudSeleccionada.fechaFormateada}
			</p>

			<p>
				<strong>Email</strong>
				{solicitudSeleccionada['Dirección de correo electrónico']}
			</p>

			<p>
				<strong>Equipo:</strong>
				{solicitudSeleccionada['EQUIPMENT / EQUIPO']}
			</p>

			<p>
				<strong>Vehículo:</strong>
				{solicitudSeleccionada['VEHICLE / VEHICULO']}
			</p>

			<p>
				<strong>Urgencia:</strong>
				{solicitudSeleccionada['URGENCY LEVEL / NIVEL DE URGENCIA']}
			</p>

			<p>
				<strong>Mantenimiento:</strong>
				{solicitudSeleccionada['MAINTENANCE REQUIRED / MANTENIMIENTO REQUERIDO']}
			</p>

			<p>
				<strong>Problema:</strong>
				{solicitudSeleccionada['PROBLEM / PROBLEMA / TEMA']}
			</p>

			<p>
				<strong>Instrucción:</strong>
				{solicitudSeleccionada['INSTRUCTION / INSTRUCCION']}
			</p>

			<p>
				<strong>Herramienta:</strong>
				{solicitudSeleccionada['HERRAMIENTA']}
			</p>

			<button
				on:click={() =>
					solicitudSeleccionada = null
				}
			>
				Cerrar
			</button>

		</div>

	</div>

{/if}

<style>
	:global(body) {
		background: #f3f4f6;
	}

	h1 {
		margin: 0 0 20px 0;
		color: #111827;
		font-size: 28px;
		font-weight: 700;
	}

	table {
		width: 100%;
		border-collapse: collapse;
		background: white;
		border-radius: 12px;
		overflow: hidden;
		box-shadow: 0 2px 10px rgba(0, 0, 0, 0.08);
	}

	thead {
		background: #f8fafc;
	}

	th {
		padding: 14px;
		text-align: left;
		font-size: 14px;
		font-weight: 700;
		color: #374151;
		border-bottom: 1px solid #e5e7eb;
	}

	td {
		padding: 14px;
		border-bottom: 1px solid #f1f5f9;
		color: #374151;
		font-size: 14px;
		vertical-align: top;
	}

	tbody tr:hover {
		background: #f9fafb;
	}

	tbody tr:last-child td {
		border-bottom: none;
	}

	.mantenimiento-container {
		padding: 20px;
		max-width: 1400px;
		margin: 0 auto;
	}

	.table-container {
		width: 100%;
		overflow-x: auto;
		border-radius: 12px;
		border: 1px solid #ddd;
		background: white;
	}

	input {
		padding: 10px;
		margin-bottom: 20px;
		width: 100%;
		max-width: 400px;
		border-radius: 8px;
		border: 1px solid #ccc;
		box-sizing: border-box;
	}

	tr {
		cursor: pointer;
	}

	tr:hover {
		background: #f9fafb;
	}

	.urgencia {
		padding: 6px 10px;
		border-radius: 8px;
		color: white;
		font-weight: 600;
		font-size: 12px;
	}

	.urgencia.high {
		background: #dc3545;
	}

	.urgencia.medium {
		background: #fd7e14;
	}

	.urgencia.low {
		background: #198754;
	}

	.overlay {
		position: fixed;
		inset: 0;
		background: rgba(0,0,0,.5);
		display: flex;
		justify-content: center;
		align-items: center;
		z-index: 1000;
	}

	.modal {
		background: white;
		padding: 24px;
		border-radius: 12px;
		width: 900px;
		max-width: 95vw;
		max-height: 90vh;
		overflow-y: auto;
		box-shadow: 0 10px 30px rgba(0,0,0,.25);
	}

	.modal h2 {
		margin-top: 0;
	}

	.modal button {
		margin-top: 20px;
		background: #2563eb;
		color: white;
		border: none;
		padding: 10px 20px;
		border-radius: 8px;
		cursor: pointer;
	}

	@media (max-width: 768px) {

		:global(main) {
			padding: 14px;
		}

		h1 {
			font-size: 24px;
		}

		table {
			display: block;
			overflow-x: auto;
			white-space: nowrap;
		}
	}
</style>