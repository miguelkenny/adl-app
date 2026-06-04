<script>
	import { onMount } from 'svelte';
	import { PUBLIC_API_URL } from '$env/static/public';
	import Loader from '$lib/components/Loader.svelte';
	
	const MOV_API =
		`${PUBLIC_API_URL}?sheet=Movimientos`;

	let loading = true;
	
	let movimientos = [];

	let obraFiltro = '';
	let fechaDesde = '';
	let fechaHasta = '';
	let usuario = null;
	let obras = [];

	onMount(async () => {

		try {

			const response =
				await fetch(MOV_API);

			movimientos =
				await response.json();

			obras = [
				...new Set(
					movimientos
						.filter(
							(m) =>
								m.Tipo === 'TRANSFERENCIA'
						)
						.map((m) => m.Destino)
						.filter(Boolean)
				)
			].sort();
			console.log(
				'Primeros movimientos:',
				movimientos.slice(0, 10)
			);
			const user =
			localStorage.getItem('user');

			if (user) {
				usuario = JSON.parse(user);
			}

		} catch (error) {

			console.error(error);

		} finally {

			loading = false;

		}
	});

	function cumpleFecha(fecha) {

		if (!fecha) return false;

		const fechaMovimiento =
			fecha.toString().split('T')[0];

		if (
			fechaDesde &&
			fechaMovimiento < fechaDesde
		) {
			return false;
		}

		if (
			fechaHasta &&
			fechaMovimiento > fechaHasta
		) {
			return false;
		}

		return true;
	}

	$: consumos =
		movimientos.filter((m) => {

			if (m.Tipo !== 'TRANSFERENCIA')
				return false;

			if (
				obraFiltro &&
				m.Destino !== obraFiltro
			)
				return false;

			// Ignorar movimientos anteriores al cambio de formato
			if (
				!m.Fecha ||
				!m.Fecha.includes('-')
			)
				return false;

			return cumpleFecha(m.Fecha);

		});

		$: console.log(
			'Transferencias filtradas:',
			consumos.length
		);

	$: totalARS =
		consumos
			.filter(i => i.Moneda === 'ARS')
			.reduce(
				(acc, i) =>
					acc + (Number(i.Total) || 0),
				0
			);

	$: totalUSD =
		consumos
			.filter(i => i.Moneda === 'USD')
			.reduce(
				(acc, i) =>
					acc + (Number(i.Total) || 0),
				0
			);

	$: resumenArticulos =
		Object.values(
			consumos.reduce(
				(acc, item) => {

					if (
						!acc[item.Articulo]
					) {

						acc[item.Articulo] = {

							articulo:
								item.Articulo,

							cantidad: 0,

							total: 0
						};
					}

					acc[item.Articulo]
						.cantidad +=
						Number(
							item.Cantidad
						);

					acc[item.Articulo]
						.total +=
						Number(
							item.Total
						);

					return acc;

				},
				{}
			)
		).sort(
			(a, b) =>
				b.total - a.total
		);
</script>

<h1>
	Envíos de Materiales por Obra
</h1>
<div class="informes-container">
	{#if loading}

		<Loader />

	{:else}

		<div class="filtros">

			<input
				type="date"
				bind:value={fechaDesde}
			/>

			<input
				type="date"
				bind:value={fechaHasta}
			/>

			<select bind:value={obraFiltro}>

				<option value="">
					Todas las obras
				</option>

				{#each obras as obra}

					<option value={obra}>
						{obra}
					</option>

				{/each}

			</select>

		</div>

		<div class="card">
			<h2>
				Total Enviado ARS
			</h2>

			<h1>
				${totalARS.toLocaleString()}
			</h1>
		</div>

		<div class="card">
			<h2>
				Total Enviado USD
			</h2>

			<h1>
				U$S {totalUSD.toLocaleString()}
			</h1>
		</div>

		<h2>
			Resumen por Artículo
		</h2>

		<table>

			<thead>

				<tr>

					<th>Artículo</th>
					<th>Cantidad</th>
					<th>Total</th>

				</tr>

			</thead>

			<tbody>

				{#each resumenArticulos as item}

					<tr>

						<td>
							{item.articulo}
						</td>

						<td>
							{item.cantidad}
						</td>

						<td>
							$
							{item.total.toLocaleString()}
						</td>

					</tr>

				{/each}

			</tbody>

		</table>

		<h2>
			Detalle de Envíos
		</h2>

		<table>

			<thead>

				<tr>

					<th>Fecha</th>
					<th>Artículo</th>
					<th>Origen</th>
					<th>Destino</th>
					<th>Referencia</th>
					<th>Cantidad</th>
					<th>Total</th>
					<th>Moneda</th>

				</tr>

			</thead>

			<tbody>

				{#each consumos as item}

					<tr>

						<td>
							{item.Fecha?.split('T')[0]}
						</td>

						<td>{item.Articulo}</td>

						<td>{item.Origen}</td>

						<td>{item.Destino}</td>
						<td>{item.Referencia}</td>
						<td>{item.Cantidad}</td>

						<td>
							$
							{Number(
								item.Total
							).toLocaleString()}
						</td>

						<td>{item.Moneda}</td>

					</tr>

				{/each}

			</tbody>

		</table>

	{/if}
</div>
<style>
	.informes-container {
		padding: 0px 14px;
	}

	h1 {
		margin-bottom: 20px;
	}

	.filtros {
		display: flex;
		gap: 10px;
		margin-bottom: 20px;
		flex-wrap: wrap;
	}

	.card {
		padding: 20px;
		background: white;
		border-radius: 10px;
		margin-bottom: 20px;
		box-shadow: 0 2px 8px rgba(0,0,0,.05);
	}

	table {
		width: 100%;
		border-collapse: collapse;
		margin-bottom: 30px;
		background: white;
	}

	th,
	td {
		padding: 10px;
		border: 1px solid #ddd;
	}

	th {
		background: #f3f4f6;
	}

	select,
	input {
		padding: 10px;
		border-radius: 8px;
		border: 1px solid #ddd;
	}
</style>