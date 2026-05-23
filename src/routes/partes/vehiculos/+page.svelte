<script>
	import { onMount } from 'svelte';
    import Loader from '$lib/components/Loader.svelte';

	let partes = [];
	let busqueda = '';
	let parteSeleccionado = null;
    let loading = true;

	const API_URL =
		'https://script.google.com/macros/s/AKfycbxNL9gf4krKLnbnPSJFhrDOj1AiL3--Zx3Lwbyq3SA3yla4zrMtSY3evhvrBggeu_qLwQ/exec?sheet=VEHICULOS';

	onMount(async () => {
		const response = await fetch(API_URL);
		partes = await response.json();
        loading = false;
	});

	$: partesFiltradas = partes
		.filter((item) => {
			const texto = busqueda.toLowerCase();

			const contenido = `
				${item['MOVIL'] || ''}
				${item['TIPO DE MANTENIMIENTO'] || ''}
				${item['DESCRIPCION DE LOS TRABAJOS REALIZADOS'] || ''}
			`
				.toLowerCase()
				.trim();

			return contenido.includes(texto);
		})
		.sort(
			(a, b) =>
				new Date(b['Marca temporal']) - new Date(a['Marca temporal'])
		);

	function alertaKm(item) {
		const odometro = Number(item['ODOMETRO']) || 0;
		const proximo = Number(item['PROXIMO SERVICE - KM.']) || 0;

		if (!proximo) return '';

		const diferencia = proximo - odometro;

		if (diferencia <= 0) return 'critico';
		if (diferencia <= 500) return 'alerta';

		return '';
	}
</script>

<h1>Partes Vehículos</h1>

<input
	type="text"
	placeholder="Buscar móvil o mantenimiento..."
	bind:value={busqueda}
/>

<p>Total visibles: {partesFiltradas.length}</p>

{#if loading}
	<Loader />
{:else}
	<table>
        <thead>
            <tr>
                <th>Fecha</th>
                <th>Móvil</th>
                <th>Tipo</th>
                <th>Odómetro</th>
				<th>Trabajos Realizados</th>
				<th>Insumos Utilizados</th>
                <th>Próx. Service KM</th>
            </tr>
        </thead>

        <tbody>
            {#each partesFiltradas as item}
                <tr on:click={() => parteSeleccionado = item}>
                    <td>{item['FECHA']}</td>
                    <td>{item['MOVIL']}</td>
                    <td>{item['TIPO DE MANTENIMIENTO']}</td>
                    <td>{item['ODOMETRO']}</td>
					<td>{item['DESCRIPCION DE LOS TRABAJOS REALIZADOS']}</td>
					<td>{item['REPUESTOS/CONSUMIBLES UTILIZADOS']}</td>
                    <td class={alertaKm(item)}>
                        {item['PROXIMO SERVICE - KM.']}
                    </td>
                </tr>
            {/each}
        </tbody>
    </table>
{/if}

{#if parteSeleccionado}
	<div class="overlay" on:click={() => parteSeleccionado = null}>
		<div class="modal" on:click|stopPropagation>

			<h2>Detalle Parte Vehículo</h2>

			<p><strong>Fecha:</strong> {parteSeleccionado['FECHA']}</p>
			<p><strong>Informado Por:</strong> {parteSeleccionado['Dirección de correo electrónico']}</p>
			<p><strong>Lugar:</strong> {parteSeleccionado['LUGAR DEL MANTENIMIENTO']}</p>
			<p><strong>Móvil:</strong> {parteSeleccionado['MOVIL']}</p>
			<p><strong>Tipo:</strong> {parteSeleccionado['TIPO DE MANTENIMIENTO']}</p>
			<p><strong>Odómetro:</strong> {parteSeleccionado['ODOMETRO']}</p>
			<p><strong>Horómetro:</strong> {parteSeleccionado['INSUMOS UTILIZADOS']}</p>

			<p><strong>Trabajos realizados:</strong></p>
			<p>{parteSeleccionado['DESCRIPCION DE LOS TRABAJOS REALIZADOS']}</p>

			<p><strong>Repuestos:</strong></p>
			<p>{parteSeleccionado['REPUESTOS/CONSUMIBLES UTILIZADOS']}</p>

			<p><strong>Personal:</strong></p>
			<p>{parteSeleccionado['PERSONAL INTERVINIENTE 1']}</p>
			<p>{parteSeleccionado['PERSONAL INTERVINIENTE 2']}</p>
			<p>{parteSeleccionado['PERSONAL INTERVINIENTE 3']}</p>

			<button on:click={() => parteSeleccionado = null}>
				Cerrar
			</button>

		</div>
	</div>
{/if}

<style>
	input {
		padding: 10px;
		width: 350px;
		margin-bottom: 20px;
		border-radius: 8px;
		border: 1px solid #ccc;
	}

	table {
		width: 100%;
		border-collapse: collapse;
		font-family: Arial;
	}

	th, td {
		padding: 12px;
		border: 1px solid #ddd;
		text-align: left;
	}

	tr {
		cursor: pointer;
	}

	tr:hover {
		background: #f8f9fa;
	}

	.alerta {
		background: #fff3cd;
		font-weight: bold;
	}

	.critico {
		background: #f8d7da;
		color: #842029;
		font-weight: bold;
	}

	.overlay {
		position: fixed;
		inset: 0;
		background: rgba(0,0,0,0.5);
		display: flex;
		justify-content: center;
		align-items: center;
	}

	.modal {
		background: white;
		padding: 30px;
		border-radius: 12px;
		width: 600px;
		max-width: 90%;
	}
</style>