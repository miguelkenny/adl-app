<script>
	import { onMount } from 'svelte';
	import Loader from '$lib/components/Loader.svelte';

	let stock = [];
	let articulos = [];
	let loading = true;
	let busqueda = '';

	const BASE_URL =
		'https://script.google.com/macros/s/AKfycbwN44dwbnjzjzR8SUV1P6DdAPosbGQn1HCzd7yxnxgV8kTi30CEpFeFqza-RAlkcWqfrg/exec';

	const STOCK_API = `${BASE_URL}?sheet=Stock`;
	const ARTICULOS_API = `${BASE_URL}?sheet=Articulos`;

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
			Number(item['Ullum'] || 0)
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

	$: stockFiltrado = stock.filter((item) =>
		item['Articulo']
			?.toLowerCase()
			.includes(busqueda.toLowerCase())
	);

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

	<table>
		<thead>
            <tr>
                <th>Código Interno</th>
                <th>Código Proveedor</th>
                <th>Artículo</th>
                <th>Contenedor Origen</th>
                <th>Albardon</th>
                <th>Casposo</th>
                <th>Barker</th>
				<th>Ullum</th>
                <th>Total</th>
                <th>Estado</th>
            </tr>
        </thead>

		<tbody>
			{#each stockFiltrado as item}
				<tr>
					<td>{getArticuloData(item['Articulo'])['Codigo Interno']}</td>

                    <td>{getArticuloData(item['Articulo'])['Codigo Proveedor']}</td>

                    <td>{item['Articulo']}</td>

                    <td>{getArticuloData(item['Articulo'])['Contenedor']}</td>

                    <td>{item['Albardon']}</td>

                    <td>{item['Casposo']}</td>

                    <td>{item['Barker']}</td>
					
					<td>{item['Ullum']}</td>

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
</style>