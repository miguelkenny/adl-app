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

		<div class="cards-resumen">
			<div class="card ars">
				<h2>
					Total Enviado ARS
				</h2>

				<h1>
					${totalARS.toLocaleString()}
				</h1>
			</div>

			<div class="card usd">
				<h2>
					Total Enviado USD
				</h2>

				<h1>
					U$S {totalUSD.toLocaleString()}
				</h1>
			</div>
		</div>
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

	{/if}
</div>
<style>
	.informes-container {
		max-width: 1400px;
		margin: 0 auto;
		padding: 20px;
	}

	h1 {
		margin: 4 0 24px 0;
		color: #111827;
		font-size: 28px;
		font-weight: 700;
	}

	h2 {
		margin-top: 30px;
		margin-bottom: 15px;
		color: #111827;
		font-size: 20px;
	}

	.card {
		padding: 22px;
		border-radius: 14px;
		color: white;
		margin-bottom: 20px;
		box-shadow: 0 4px 12px rgba(0,0,0,.08);
	}

	.card h2 {
		margin: 0;
		font-size: 16px;
		font-weight: 600;
	}

	.card h1 {
		margin-top: 10px;
		margin-bottom: 0;
		font-size: 32px;
		color: white;
	}

	.cards-resumen {
		display: flex;
		gap: 20px;
		margin-bottom: 24px;
		flex-wrap: wrap;
	}

	.cards-resumen .card {
		flex: 1 1 300px;
	}

	.ars {
		background: #198754;
	}

	.usd {
		background: #2563eb;
	}

	table {
		width: 100%;
		border-collapse: collapse;
		background: white;
		margin-bottom: 30px;
		box-shadow: 0 2px 8px rgba(0,0,0,.05);
		border-radius: 12px;
		overflow: hidden;
	}

	thead {
		background: #9b9c9c;
	}

	th {
		padding: 14px;
		text-align: left;
		font-weight: 600;
		border-bottom: 1px solid #e5e7eb;
	}

	td {
		padding: 12px;
		border-bottom: 1px solid #f3f4f6;
		color: #374151;
		font-size: 12px;
	}

	tbody tr:hover {
		background: #f9fafb;
	}

	.filtros {
		display: flex;
		gap: 12px;
		margin-bottom: 24px;
		flex-wrap: wrap;
	}

	.filtros input,
	.filtros select {
		padding: 12px;
		border: 1px solid #d1d5db;
		border-radius: 10px;
		background: white;
		font-size: 14px;
		min-width: 220px;
		box-sizing: border-box;
	}

	.filtros input:focus,
	.filtros select:focus {
		outline: none;
		border-color: #2563eb;
		box-shadow: 0 0 0 3px rgba(37,99,235,.15);
	}
</style>