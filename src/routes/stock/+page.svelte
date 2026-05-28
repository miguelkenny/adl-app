<script>
	import { onMount } from 'svelte';
	import Loader from '$lib/components/Loader.svelte';

	let stock = [];
	let articulos = [];
	let loading = true;
	let busqueda = '';
	let paginaActual = 1;
	let itemsPorPagina = 25;

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
			Number(item['Ullum'] || 0) +
			Number(item['Taller Albardon'] || 0)
		);
	}

	function getStockMinimo(nombreArticulo) {
		const articulo = articulos.find(
			(item) => item['Nombre del elemento'] === nombreArticulo
		);

		return Number(articulo?.['Stock Minimo']) || 0;
	}

	function getEstado(item) {
		const total = getTotal(item);
		const minimo = getStockMinimo(item['Articulo']);

		if (total === 0) return 'agotado';
		if (total <= minimo) return 'reponer';
		return 'ok';
	}

	$: stockFiltrado = stock.filter((item) => {

		const busquedaLower = busqueda.toLowerCase();

		const articulo = item['Articulo']?.toLowerCase() || '';

		const articuloData = getArticuloData(item['Articulo']);

		const codigoInterno =
			String(articuloData['Codigo Interno'] || '').toLowerCase();

		const codigoProveedor =
			String(articuloData['Codigo Proveedor'] || '').toLowerCase();

		return (
			articulo.includes(busquedaLower) ||
			codigoInterno.includes(busquedaLower) ||
			codigoProveedor.includes(busquedaLower)
		);
	});

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

    function getArticuloData(nombreArticulo) {
        return (
            articulos.find(
                (item) =>
                    item['Nombre del elemento'] === nombreArticulo
            ) || {}
        );
    }
</script>

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

<input
	type="text"
	placeholder="Buscar artículo..."
	bind:value={busqueda}
/>

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
					<th>Marca</th>
					<th>Contenedor Origen</th>
					<th>Albardon</th>
					<th>Casposo</th>
					<th>Barker</th>
					<th>Ullum</th>
					<th>Taller Albardon</th>
					<th>Total</th>
					<th>Estado</th>
				</tr>
			</thead>

			<tbody>
				{#each stockPaginado as item}
					<tr>
						<td>{getArticuloData(item['Articulo'])['Codigo Interno']}</td>

						<td>{getArticuloData(item['Articulo'])['Codigo Proveedor']}</td>

						<td>{item['Articulo']}</td>

						<td>{getArticuloData(item['Articulo'])['Marca']}</td>

						<td>{getArticuloData(item['Articulo'])['Contenedor']}</td>

						<td>{item['Albardon']}</td>

						<td>{item['Casposo']}</td>

						<td>{item['Barker']}</td>
						
						<td>{item['Ullum ALFA']}</td>

						<td>{item['Taller Albardon']}</td>

						<td>{getTotal(item)}</td>

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

<style>
	h1 {
		margin-bottom: 20px;
	}

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

	th,
	td {
		padding: 12px;
		border: 1px solid #ddd;
		text-align: left;
	}

	tr:hover {
		background: #f8f9fa;
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

</style>