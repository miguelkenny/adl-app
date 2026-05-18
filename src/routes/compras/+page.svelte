<script>
	import { onMount } from 'svelte';
    import Loader from '$lib/components/Loader.svelte';

	let solicitudes = [];
	let filtroEstado = 'Todos';
	let busqueda = '';
	let pedidoSeleccionado = null;
    let loading = true;

	const API_URL = 'https://script.google.com/macros/s/AKfycbxzgWF1fHx0_yuIFB0HaJ3oo1pGU8wW_2Wyh-B0-jOo0aO_R-4xnJxSNm5LF9VDzKVwYQ/exec';

	onMount(async () => {
		const response = await fetch(API_URL);
		solicitudes = await response.json();
        loading = false;
	});

	$: solicitudesFiltradas = solicitudes
	.filter((item) => {
		const coincideEstado =
			filtroEstado === 'Todos' || item['ESTADO'] === filtroEstado;

		const textoBusqueda = busqueda.toLowerCase().trim();

		const textoCompleto = `
			${item['DESCRIPCION'] || ''}
			${item['EQUIPO'] || ''}
			${item['MARCA'] || ''}
			${item['CODIGO / NUMERO DE PARTE'] || ''}
		`
			.toLowerCase()
			.trim();

		const coincideBusqueda = textoCompleto.includes(textoBusqueda);

		return coincideEstado && coincideBusqueda;
	})
	.sort((a, b) => {
		const ordenEstados = {
			'Pendiente': 1,
			'Presupuestando': 2,
			'En proceso compra': 3,
			'Comprado esperando llegada': 4,
			'Completado': 5,
			'Cancelado o Suspendido': 6
		};

		const ordenA = ordenEstados[a['ESTADO']] || 99;
		const ordenB = ordenEstados[b['ESTADO']] || 99;

		if (ordenA !== ordenB) return ordenA - ordenB;

		const fechaA = new Date(a.timestampOriginal).getTime();
		const fechaB = new Date(b.timestampOriginal).getTime();

		// completados y cancelados: más nuevos primero
		if (
			a['ESTADO'] === 'Completado' ||
			a['ESTADO'] === 'Cancelado o Suspendido'
		) {
			return fechaB - fechaA;
		}

		// resto: más viejos primero
		return fechaA - fechaB;
	});

	$: pendientes = solicitudes.filter(
			(item) => item['ESTADO'] === 'Pendiente'
		).length;

		$: presupuestando = solicitudes.filter(
			(item) => item['ESTADO'] === 'Presupuestando'
		).length;

		$: enProceso = solicitudes.filter(
			(item) => item['ESTADO'] === 'En proceso compra'
		).length;

		$: completados = solicitudes.filter(
			(item) => item['ESTADO'] === 'Completado'
		).length;

	async function actualizarEstado(item, nuevoEstado) {
		try {
			const response = await fetch(API_URL, {
				method: 'POST',
				body: JSON.stringify({
					timestamp: item.timestampOriginal,
					estado: nuevoEstado
				})
			});

			const result = await response.json();
			console.log(result);

			if (result.success) {
				item['ESTADO'] = nuevoEstado;

				// fuerza reactividad
				solicitudes = [...solicitudes];
			}
		} catch (error) {
			console.error(error);
		}
	}

	function diasTranscurridos(fecha) {
		const inicio = new Date(fecha);
		const hoy = new Date();

		const diferencia = hoy - inicio;

		return Math.floor(diferencia / (1000 * 60 * 60 * 24));
	}
</script>

<h1>ADL Compras</h1>

<div class="cards">

	<div class="card pendientes" on:click={() => filtroEstado = 'Pendiente'}>
		<h3>Pendientes</h3>
		<p>{pendientes}</p>
	</div>

	<div class="card presupuestando" on:click={() => filtroEstado = 'Presupuestando'}>
		<h3>Presupuestando</h3>
		<p>{presupuestando}</p>
	</div>

	<div class="card proceso" on:click={() => filtroEstado = 'En proceso compra'}>
		<h3>En compra</h3>
		<p>{enProceso}</p>
	</div>

	<div class="card completados" on:click={() => filtroEstado = 'Completado'}>
		<h3>Completados</h3>
		<p>{completados}</p>
	</div>

</div>

<select bind:value={filtroEstado}>
	<option>Todos</option>
	<option>Pendiente</option>
	<option>Presupuestando</option>
	<option>En proceso compra</option>
	<option>Comprado esperando llegada</option>
	<option>Completado</option>
	<option>Cancelado o suspendido</option>
</select>

<input
	type="text"
	placeholder="Buscar por equipo, descripción, marca o código..."
	bind:value={busqueda}
/>

<p>Total visibles: {solicitudesFiltradas.length}</p>
<button
	on:click={() => {
		filtroEstado = 'Todos';
		busqueda = '';
	}}
>
	Limpiar filtros
</button>
{#if loading}
	<Loader />
{:else}
	<table border="1">
	<thead>
		<tr>
			<th>Fecha</th>
			<th>Equipo</th>
			<th>Especificacion</th>
			<th>Descripcion</th>
			<th>Codigo / N° de Parte</th>
			<th>Días</th>
			<th>Estado</th>
		</tr>
	</thead>
    
	<tbody>
		{#each solicitudesFiltradas as item}
			<tr on:click={() => pedidoSeleccionado = item}>
				<td>{item.fechaFormateada}</td>
				<td>{item['EQUIPO']}</td>
				<td>{item['ESPECIFICACION']}</td>
				<td>{item['DESCRIPCION']}</td>
				<td>{item['CODIGO / NUMERO DE PARTE']}</td>
				<td>
					{#if item['ESTADO'] === 'Completado' || item['ESTADO'] === 'Cancelado o Suspendido'}
						—
					{:else}
						{diasTranscurridos(item.timestampOriginal)}
					{/if}
				</td>
				<td>
					<select
						class={`estado-select ${(item['ESTADO'] || 'Pendiente')
								.trim()
								.replace(/\s+/g, '-')
								.toLowerCase()}`}
						value={item['ESTADO'] || item['Pendiente']}
						on:change={(e) => actualizarEstado(item, e.target.value)}
					>
						<option>Pendiente</option>
						<option>Presupuestando</option>
						<option>En proceso compra</option>
						<option>Comprado esperando llegada</option>
						<option>Completado</option>
						<option>Cancelado o Suspendido</option>
					</select>
				</td>
			</tr>
		{/each}
	</tbody>
</table>
{/if}

{#if pedidoSeleccionado}
	<div class="overlay" on:click={() => pedidoSeleccionado = null}>
		<div class="modal" on:click|stopPropagation>

			<h2>Detalle Solicitud</h2>

			<p><strong>Fecha:</strong> {pedidoSeleccionado.fechaFormateada}</p>
			<p><strong>Equipo:</strong> {pedidoSeleccionado['EQUIPO']}</p>
			<p><strong>Lugar:</strong> {pedidoSeleccionado['LUGAR']}</p>
			<p><strong>Clasificación:</strong> {pedidoSeleccionado['CLASIFICACION']}</p>
			<p><strong>Especificación:</strong> {pedidoSeleccionado['ESPECIFICACION']}</p>
			<p><strong>Marca:</strong> {pedidoSeleccionado['MARCA']}</p>
			<p><strong>Código:</strong> {pedidoSeleccionado['CODIGO / NUMERO DE PARTE']}</p>
			<p><strong>Cantidad:</strong> {pedidoSeleccionado['CANTIDAD']}</p>
			<p><strong>Descripción:</strong> {pedidoSeleccionado['DESCRIPCION']}</p>
			<p><strong>Estado:</strong> {pedidoSeleccionado['ESTADO']}</p>

			{#if pedidoSeleccionado['IMAGEN / FOTO']}
				<img
					src={`https://drive.google.com/uc?export=view&id=${
						pedidoSeleccionado['IMAGEN / FOTO'].split('id=')[1]
					}`}
					alt="Imagen del pedido"
					class="imagen-pedido"
				/>
			{/if}

			<button on:click={() => pedidoSeleccionado = null}>
				Cerrar
			</button>

		</div>
	</div>
{/if}
<style>
	h1 {
		font-family: Arial, sans-serif;
		margin-bottom: 20px;
	}

	table {
		width: 100%;
		border-collapse: collapse;
		font-family: Arial, sans-serif;
	}

	th,
	td {
		padding: 12px;
		border: 1px solid #ddd;
		text-align: left;
	}

	th {
		background: #f5f5f5;
	}

	.estado {
		padding: 6px 12px;
		border-radius: 8px;
		font-weight: bold;
		color: white;
		display: inline-block;
	}

	.pendiente {
	background: #dc3545;
	}

	.presupuestando {
		background: #fd7e14;
	}

	.en-proceso-compra {
		background: #0d6efd;
	}

	.comprado-esperando-llegada {
		background: #6f42c1;
	}

	.completado {
		background: #198754;
	}

	.cancelado-o-suspendido {
		background: #6c757d;
	}

	select {
		padding: 10px;
		margin-bottom: 20px;
		width: 350px;
		border-radius: 8px;
		border: 1px solid #ccc;
		font-size: 14px;
	}

	input {
		padding: 10px;
		margin-left: 0px;
		margin-bottom: 20px;
		width: 350px;
		border-radius: 8px;
		border: 1px solid #ccc;
		font-size: 14px;
	}

	.cards {
	display: flex;
	gap: 20px;
	margin-bottom: 20px;
	flex-wrap: wrap;
	}

	.card {
		padding: 20px;
		border-radius: 12px;
		color: white;
		min-width: 180px;
		font-family: Arial, sans-serif;
	}

	.card h3 {
		margin: 0;
		font-size: 16px;
	}

	.card p {
		font-size: 28px;
		font-weight: bold;
		margin-top: 10px;
		margin-bottom: 0;
	}

	.pendientes {
		background: #dc3545;
	}

	.presupuestando {
		background: #fd7e14;
	}

	.proceso {
		background: #0d6efd;
	}

	.completados {
		background: #198754;
	}

	.card {
		cursor: pointer;
		transition: transform 0.1s ease;
	}

	.card:hover {
		transform: scale(1.05);
	}

	tr {
	cursor: pointer;
	}

	tr:hover {
		background: #f8f9fa;
	}

	.overlay {
		position: fixed;
		inset: 0;
		background: rgba(0, 0, 0, 0.5);
		display: flex;
		justify-content: center;
		align-items: center;
		z-index: 1000;
	}

	.modal {
		background: white;
		padding: 30px;
		border-radius: 12px;
		width: 500px;
		max-width: 90%;
		font-family: Arial, sans-serif;
	}

	.modal h2 {
		margin-top: 0;
	}

	.modal button {
		margin-top: 20px;
		padding: 10px 20px;
		border: none;
		background: #0d6efd;
		color: white;
		border-radius: 8px;
		cursor: pointer;
	}

	.imagen-pedido {
		width: 100%;
		margin-top: 20px;
		border-radius: 10px;
		border: 1px solid #ddd;
	}

	.alerta {
		background: #fff3cd;
		font-weight: bold;
	}

	.critico {
		background: #f8d7da;
		font-weight: bold;
		color: #842029;
	}

	.estado-select {
	padding: 8px 12px;
	border: none;
	border-radius: 8px;
	color: white;
	font-weight: bold;
	cursor: pointer;
}

/* estados */
.Pendiente {
	background: #dc3545;
}

.Presupuestando {
	background: #fd7e14;
}

.En-proceso-compra {
	background: #0d6efd;
}

.Comprado-esperando-llegada {
	background: #6f42c1;
}

.Completado {
	background: #198754;
}

.Cancelado-o-Suspendido {
	background: #6c757d;
}

button {
	padding: 10px 10px;
	border: none;
	border-radius: 8px;
	background: #6c757d;
	color: white;
	cursor: pointer;
	width: 350px;
	margin-bottom: 10px;
}
</style>