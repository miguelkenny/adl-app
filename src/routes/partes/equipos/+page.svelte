<script>
	import { onMount } from 'svelte';
    import Loader from '$lib/components/Loader.svelte';

	let partes = [];
	let busqueda = '';
	let parteSeleccionado = null;
    let loading = true;

	const API_URL = 'https://script.google.com/macros/s/AKfycbxNL9gf4krKLnbnPSJFhrDOj1AiL3--Zx3Lwbyq3SA3yla4zrMtSY3evhvrBggeu_qLwQ/exec?sheet=EQUIPOS';

	onMount(async () => {
		const response = await fetch(API_URL);
		partes = await response.json();
        loading=false;
	});

	$: partesFiltradas = partes
	.filter((item) => {
		const texto = busqueda.toLowerCase();

		const contenido = `
			${item['EQUIPO'] || ''}
			${item['TIPO DE MANTENIMIENTO'] || ''}
			${item['PROBLEMA / SINTOMA'] || ''}
		`
			.toLowerCase();

		return contenido.includes(texto);
	})
	.sort((a, b) =>
		new Date(b['Marca temporal']) - new Date(a['Marca temporal'])
	);

	function alertaHoras(actual, proximo) {
		const actualNum = Number(actual) || 0;
		const proximoNum = Number(proximo) || 0;

		if (!proximoNum) return '';

		const diferencia = proximoNum - actualNum;

		if (diferencia <= 0) return 'critico';
		if (diferencia <= 100) return 'alerta';

		return '';
	}
</script>

<h1>Partes de Equipos</h1>

<input
	type="text"
	placeholder="Buscar equipo o problema..."
	bind:value={busqueda}
/>

<p>Total visibles: {partesFiltradas.length}</p>

{#if loading}
	<Loader />
{:else}
	<table border="1">
        <thead>
            <tr>
                <th>Fecha</th>
				<th>Email</th>
                <th>Equipo</th>
                <th>Problema/Sintoma</th>
                <th>Motor</th>
                <th>Martillo</th>
                <th>Próx. Motor</th>
                <th>Próx. Martillo</th>
            </tr>
        </thead>

        <tbody>
            {#each partesFiltradas as item}
                <tr on:click={() => parteSeleccionado = item}>
                    <td>{item['FECHA']}</td>
					<td>{item['Dirección de correo electrónico']}</td>
                    <td>{item['EQUIPO']}</td>
                    <td>{item['PROBLEMA / SINTOMA']}</td>
                    <td>{item['MOTOR']}</td>
                    <td>{item['MARTILLO']}</td>

                    <td class={alertaHoras(item['MOTOR'], item['PROXIMO SERVICE - HORAS MOTOR'])}>
                        {item['PROXIMO SERVICE - HORAS MOTOR']}
                    </td>

                    <td class={alertaHoras(item['MARTILLO'], item['PROXIMO SERVICE - HORAS MARTILLO'])}>
                        {item['PROXIMO SERVICE - HORAS MARTILLO']}
                    </td>
                </tr>
            {/each}
        </tbody>
    </table>
{/if}

{#if parteSeleccionado}
	<div class="overlay" on:click={() => parteSeleccionado = null}>
		<div class="modal" on:click|stopPropagation>

			<h2>Detalle Parte Equipo</h2>

			<p><strong>Fecha:</strong> {parteSeleccionado['FECHA']}</p>
			<p><strong>Informado Por:</strong> {parteSeleccionado['Dirección de correo electrónico']}</p>
			<p><strong>Equipo:</strong> {parteSeleccionado['EQUIPO']}</p>
			<p><strong>Ubicación:</strong> {parteSeleccionado['UBICACION']}</p>
			<p><strong>Problema:</strong> {parteSeleccionado['PROBLEMA / SINTOMA']}</p>
			<p><strong>Sector:</strong> {parteSeleccionado['SECTOR / CIRCUITO / PARTE']}</p>
			<p><strong>Tipo:</strong> {parteSeleccionado['TIPO DE MANTENIMIENTO']}</p>

			<p><strong>Trabajos realizados:</strong></p>
			<p>{parteSeleccionado['TRABAJOS REALIZADOS']}</p>

			<p><strong>Insumos:</strong></p>
			<p>{parteSeleccionado['INSUMOS UTILIZADOS']}</p>

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