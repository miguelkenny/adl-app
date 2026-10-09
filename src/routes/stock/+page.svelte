<script>
	import { onMount } from 'svelte';
	import Loader from '$lib/components/Loader.svelte';

	let stock = [];
	let articulos = [];
	let loading = true;
	let busqueda = '';
	let almacenFiltro = '';
	let contenedorFiltro = '';
	let paginaActual = 1;
	let itemsPorPagina = 25;
	let usuario = null;

	import { PUBLIC_API_URL } from '$env/static/public';

	const STOCK_API = `${PUBLIC_API_URL}?sheet=Stock`;
	const ARTICULOS_API = `${PUBLIC_API_URL}?sheet=Articulos`;

	onMount(async () => {
		try {
			const [stockRes, articulosRes] = await Promise.all([
				fetch(STOCK_API),
				fetch(ARTICULOS_API)
			]);

			stock = await stockRes.json();
			articulos = await articulosRes.json();

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

	function getTotal(item) {
		return (
			Number(item['Albardon'] || 0) +
			Number(item['Casposo'] || 0) +
			Number(item['Barker'] || 0) +
			Number(item['Ullum ALFA'] || 0) +
			Number(item['Taller Albardon'] || 0)
		);
	}

	function getStockVisible(item) {

		if (!almacenFiltro) {
			return getTotal(item);
		}

		return Number(
			item[almacenFiltro] || 0
		);
	}

	
	function getStockMinimo(itemStock) {
		const articulo = getArticuloData(itemStock);
		return Number(articulo['Stock Minimo']) || 0;
	}

	function getEstado(item) {
		const total = getTotal(item);
		const minimo = getStockMinimo(item);

		if (total === 0) return 'agotado';
		if (total <= minimo) return 'reponer';
		return 'ok';
	}

	function normalizar(valor) {
		return String(valor ?? '')
			.trim()
			.toLowerCase()
			.replace(/\s+/g, ' ');
	}
	
	$: stockFiltrado = stock.filter((item) => {
		const busquedaNormalizada = normalizar(busqueda);

		const articulo = normalizar(item['Articulo']);
		const articuloData = getArticuloData(item);

		const codigoInterno = normalizar(
			articuloData['Codigo Interno']
		);

		const codigoProveedor = normalizar(
			articuloData['Codigo Proveedor']
		);

		const contenedor = normalizar(
			articuloData['Contenedor']
		);

		const coincideBusqueda =
			!busquedaNormalizada ||
			articulo.includes(busquedaNormalizada) ||
			codigoInterno.includes(busquedaNormalizada) ||
			codigoProveedor.includes(busquedaNormalizada);

		const coincideContenedor =
			!contenedorFiltro ||
			contenedor === normalizar(contenedorFiltro);

		const coincideAlmacen =
			!almacenFiltro ||
			Number(item[almacenFiltro] || 0) > 0;

		return (
			coincideBusqueda &&
			coincideContenedor &&
			coincideAlmacen
		);
	});

	$: if (!loading && stock.length && articulos.length) {
		console.log(
			'Filas Stock con código 040214-010430:',
			stock.filter(item =>
				String(item['Codigo Proveedor'] ?? '').trim() === '040214-010430' ||
				String(getArticuloData(item['Articulo'])['Codigo Proveedor'] ?? '').trim() === '040214-010430'
			)
		);

		console.log(
			'Filas Articulos con código 040214-010430:',
			articulos.filter(item =>
				String(item['Codigo Proveedor'] ?? '').trim() === '040214-010430'
			)
		);
	}

	$: totalPaginas = Math.ceil(
		stockFiltrado.length / itemsPorPagina
	);

	$: stockPaginado = stockFiltrado.slice(
		(paginaActual - 1) * itemsPorPagina,
		paginaActual * itemsPorPagina
	);

	$: if (busqueda) {
		paginaActual = 1;
	}

    
	function normalizarCodigo(valor) {
		return String(valor ?? '').trim();
	}

	function getArticuloData(itemStock) {
		const codigoInterno = normalizarCodigo(
			itemStock?.['Codigo Interno']
		);

		return (
			articulos.find(
				(item) =>
					normalizarCodigo(item['Codigo Interno']) === codigoInterno
			) || {}
		);
	}

	$: contenedores = [
		...new Set(
			articulos
				.map(a => a['Contenedor'])
				.filter(Boolean)
		)
	].sort();

</script>
<div class="stock-container">
<div class="header">
	<h1>Stock General</h1>

	<div class="acciones">
		<a href="/stock/consumibles" class="consumibles-btn">
			Ver Consumibles
		</a>

		<a href="/stock/nuevo" class="nuevo-btn">
			+ Nuevo Movimiento
		</a>

		<a href="/stock/articulos/nuevo" class="articulo-btn">
			+ Agregar Artículo
		</a>
	</div>
</div>

<div class="filtros">

	<input
		type="text"
		placeholder="Buscar artículo..."
		bind:value={busqueda}
	/>

	<select bind:value={almacenFiltro}>

		<option value="">
			Todos los almacenes
		</option>

		<option value="Albardon">
			Albardón
		</option>

		<option value="Casposo">
			Casposo
		</option>

		<option value="Barker">
			Barker
		</option>

		<option value="Ullum ALFA">
			Ullum ALFA
		</option>

		<option value="Taller Albardon">
			Taller Albardón
		</option>

	</select>

	<select bind:value={contenedorFiltro}>

		<option value="">
			Todos los contenedores
		</option>

		{#each contenedores as contenedor}

			<option value={contenedor}>
				{contenedor}
			</option>

		{/each}

	</select>

</div>

{#if loading}
	<Loader
		mensaje="Cargando stock..."
		subtitulo="Consultando almacenes"
	/>
{:else}
	<p>Total artículos: {stockFiltrado.length}</p>

	<div class="table-container">
		<table>
			<thead>
				<tr>
					<th>Código Interno</th>
					<th>Código Proveedor</th>
					<th>Artículo</th>
					<th>Nom. Proveedor</th>
					<th>Marca</th>
					<th>Contenedor Origen</th>
					{#if !almacenFiltro}

						<th>Albardon</th>
						<th>Casposo</th>
						<th>Barker</th>
						<th>Ullum ALFA</th>
						<th>Taller Albardon</th>
						<th>Total</th>

					{:else}

						<th>{almacenFiltro}</th>

					{/if}
					
					<th>Estado</th>
				</tr>
			</thead>

			<tbody>
				{#each stockPaginado as item}
					<tr>
						<td>{getArticuloData(item)['Codigo Interno']}</td>

						<td>{getArticuloData(item)['Codigo Proveedor']}</td>

						<td>{item['Articulo']}</td>

						<td>{getArticuloData(item)['Proveedor']}</td>

						<td>{getArticuloData(item)['Marca']}</td>

						<td>{getArticuloData(item)['Contenedor']}</td>

						{#if !almacenFiltro}

							<td>{item['Albardon']}</td>

							<td>{item['Casposo']}</td>

							<td>{item['Barker']}</td>

							<td>{item['Ullum ALFA']}</td>

							<td>{item['Taller Albardon']}</td>

							<td>{getTotal(item)}</td>

						{:else}

							<td>{item[almacenFiltro] || 0}</td>

						{/if}

						<td>
							<span class={getEstado(item)}>
								{#if getEstado(item) === 'agotado'}
									Agotado
								{:else if getEstado(item) === 'reponer'}
									Reponer
								{:else}
									OK
								{/if}
							</span>
						</td>
					</tr>
				{/each}
			</tbody>
		</table>
	</div>
	<div class="paginacion">

		<button
			on:click={() => paginaActual--}
			disabled={paginaActual === 1}
		>
			← Anterior
		</button>

		<span>
			Página {paginaActual} de {totalPaginas}
		</span>

		<button
			on:click={() => paginaActual++}
			disabled={paginaActual === totalPaginas}
		>
			Siguiente →
		</button>

	</div>
{/if}
</div>

<style>
	.stock-container {
		padding: 0 14px;
	}

	h1 {
		margin-bottom: 20px;
	}

	input {
		padding: 10px;
		margin-bottom: 20px;
		width: 100%;
		max-width: 500px;

		border-radius: 8px;
		border: 1px solid #ccc;
		box-sizing: border-box;
	}

	table {
		width: 100%;
		border-collapse: collapse;
		font-family: Arial, sans-serif;
		text-align: center;
		table-layout: fixed;
	}

	th{
		padding: 8px 0;
		background: #9c9b9b;
	}
	
	tr:hover {
		background: #f8f9fa;
	}

	thead th {
		position: sticky;
		top: 0;
		background: #a8a8a8;
		z-index: 50;
		box-shadow: 0 2px 4px rgba(0,0,0,.08);
	}

	tbody {
		text-align: center;
	}

	td{
		font-size: 12px;
	}
	.ok {
		background: #d1e7dd;
		color: #0f5132;
		padding: 6px 10px;
		border-radius: 8px;
		font-weight: bold;
	}

	.reponer {
		background: #fff3cd;
		color: #664d03;
		padding: 6px 10px;
		border-radius: 8px;
		font-weight: bold;
	}

	.agotado {
		background: #f8d7da;
		color: #842029;
		padding: 6px 10px;
		border-radius: 8px;
		font-weight: bold;
	}

    .header {
        display: flex;
        justify-content: space-between;
        align-items: center;
        margin-bottom: 20px;
    }

    .nuevo-btn {
        background: #198754;
        color: white;
        padding: 10px 16px;
        border-radius: 8px;
        text-decoration: none;
        font-weight: bold;
    }

    .nuevo-btn:hover {
        opacity: 0.9;
    }

    .acciones {
		display: flex;
		gap: 12px;
		align-items: center;
	}

	.consumibles-btn {
		background: #2563eb;
		color: white;
		padding: 10px 16px;
		border-radius: 8px;
		text-decoration: none;
		font-weight: bold;
	}

	.consumibles-btn:hover {
		opacity: 0.9;
	}

	.nuevo-btn {
		background: #198754;
		color: white;
		padding: 10px 16px;
		border-radius: 8px;
		text-decoration: none;
		font-weight: bold;
	}

	.nuevo-btn:hover {
		opacity: 0.9;
	}

	.articulo-btn {
		background: #7c3aed;
		color: white;
		padding: 10px 16px;
		border-radius: 8px;
		text-decoration: none;
		font-weight: bold;
	}

	.articulo-btn:hover {
		opacity: 0.9;
	}

	.table-container {
		width: 100%;
		overflow-x: auto;
		overflow-y: auto;
		max-height: 75vh;

		border: 1px solid #ddd;
		border-radius: 12px;
	}

	.paginacion {
		display: flex;
		justify-content: center;
		align-items: center;
		gap: 16px;
		margin-top: 20px;
		padding-bottom: 20px;
	}

	.paginacion button {
		background: #2563eb;
		color: white;
		border: none;
		padding: 10px 16px;
		border-radius: 8px;
		cursor: pointer;
		font-weight: bold;
	}

	.paginacion button:disabled {
		opacity: 0.5;
		cursor: not-allowed;
	}

	.filtros {
		display: flex;
		gap: 12px;
		flex-wrap: wrap;
		margin-bottom: 20px;
	}

	.filtros input,
	.filtros select {
		padding: 8px;
		border-radius: 8px;
		border: 1px solid #ccc;
		min-width: 220px;
		box-sizing: border-box;
	}

	.filtros select {
		background: white;
	}

</style>