<script>
	import { PUBLIC_API_URL } from '$env/static/public';

	const STOCK_API = `${PUBLIC_API_URL}?sheet=Stock`;
	const ARTICULOS_API = `${PUBLIC_API_URL}?sheet=Articulos`;
	
	let stock = [];
	let articulos = [];
	let loading = true;
	console.log(PUBLIC_API_URL);
	async function cargarDatos() {
		try {
			const [stockRes, articulosRes] = await Promise.all([
				fetch(STOCK_API),
				fetch(ARTICULOS_API)
			]);

			stock = await stockRes.json();
			articulos = await articulosRes.json();
		} catch (error) {
			console.error('Error cargando consumibles:', error);
		} finally {
			loading = false;
		}
	}

	function getTotalStock(item) {
		return (
			Number(item['Albardon'] || 0) +
			Number(item['Casposo'] || 0) +
			Number(item['Barker'] || 0)
		);
	}

	function agruparConsumibles() {
		const resultado = {};

		stock.forEach((item) => {
			const articulo = articulos.find(
				(a) => a['Nombre del elemento'] === item['Articulo']
			);

			if (!articulo || !articulo['Consumible']) return;

			const consumible = articulo['Consumible'];

			if (!resultado[consumible]) {
				resultado[consumible] = {
					consumible,
					Albardon: 0,
					Casposo: 0,
					Barker: 0,
					Ullum:0,
					Total: 0
				};
			}

			resultado[consumible].Albardon += Number(item['Albardon'] || 0);
			resultado[consumible].Casposo += Number(item['Casposo'] || 0);
			resultado[consumible].Barker += Number(item['Barker'] || 0);
			resultado[consumible].Ullum += Number(item['Ullum'] || 0);
			resultado[consumible].Total += getTotalStock(item);
		});

		return Object.values(resultado).sort((a, b) =>
			a.consumible.localeCompare(b.consumible)
		);
	}

	function totalConsumibles() {
		return agruparConsumibles().length;
	}

	function stockCritico() {
		return agruparConsumibles().filter((item) => item.Total <= 10).length;
	}

	cargarDatos();
</script>

<div class="container">
	{#if loading}
		<div class="card">
			<p>Cargando consumibles...</p>
		</div>
	{:else}
		<h1>Stock de Consumibles</h1>

		<div class="stats">
			<div class="stat-card">
				<h3>{totalConsumibles()}</h3>
				<p>Consumibles activos</p>
			</div>

			<div class="stat-card warning">
				<h3>{stockCritico()}</h3>
				<p>Stock crítico</p>
			</div>
		</div>

		<div class="card">
			<table>
				<thead>
					<tr>
						<th>Consumible</th>
						<th>Albardon</th>
						<th>Casposo</th>
						<th>Barker</th>
						<th>Ullum ALFA</th>
						<th>Total</th>
					</tr>
				</thead>

				<tbody>
					{#each agruparConsumibles() as item}
						<tr>
							<td>{item.consumible}</td>
							<td>{item.Albardon}</td>
							<td>{item.Casposo}</td>
							<td>{item.Barker}</td>
							<td>{item.Ullum}</td>
							<td class:itemCritico={item.Total <= 10}>
								{item.Total}
							</td>
						</tr>
					{/each}
				</tbody>
			</table>
		</div>
	{/if}
</div>

<style>
	.container {
		max-width: 1300px;
		margin: 0 auto;
		padding: 24px;
	}

	h1 {
		font-size: 28px;
		margin-bottom: 24px;
		color: #1f2937;
	}

	p {
		font-size: 16px;
		color: #6b7280;
	}

	.card {
		background: white;
		padding: 24px;
		border-radius: 14px;
		box-shadow: 0 4px 14px rgba(0, 0, 0, 0.08);
		margin-bottom: 24px;
	}

	.stats {
		display: flex;
		gap: 20px;
		margin-bottom: 24px;
	}

	.stat-card {
		background: white;
		padding: 20px;
		border-radius: 14px;
		box-shadow: 0 4px 14px rgba(0, 0, 0, 0.08);
		min-width: 220px;
	}

	.stat-card h3 {
		font-size: 32px;
		margin: 0;
		color: #111827;
	}

	.stat-card p {
		margin-top: 8px;
		font-size: 14px;
		color: #6b7280;
	}

	.warning h3 {
		color: #dc2626;
	}

	table {
		width: 100%;
		border-collapse: collapse;
	}

	thead {
		background: #111827;
		color: white;
	}

	th {
		padding: 14px;
		text-align: left;
		font-size: 13px;
		text-transform: uppercase;
		letter-spacing: 0.05em;
	}

	td {
		padding: 14px;
		border-bottom: 1px solid #e5e7eb;
		font-size: 14px;
		color: #374151;
	}

	tbody tr:hover {
		background: #f9fafb;
	}

	tbody tr:last-child td {
		border-bottom: none;
	}

	.itemCritico {
		color: #dc2626;
		font-weight: bold;
	}
</style>