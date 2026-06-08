<script>
	import { onMount } from 'svelte';
	import { PUBLIC_API_URL } from '$env/static/public';

	let movimientos = [];
	let loading = true;

	let busqueda = '';

	let fechaDesde = '';
	let fechaHasta = '';

	let tipo = 'Todos';

    let movimientoSeleccionado = null;

    let tipoFiltro = '';

	onMount(async () => {

		try {

			const response = await fetch(
				`${PUBLIC_API_URL}?sheet=Movimientos`
			);

			movimientos = await response.json();

			movimientos.sort(
				(a, b) =>
					new Date(b.Fecha) -
					new Date(a.Fecha)
			);

		} finally {

			loading = false;
		}
	});

	function formatFecha(fecha) {

		if (!fecha) return '';

		return new Date(
			fecha
		).toLocaleDateString('es-AR');
	}

	$: movimientosFiltrados =
		movimientos.filter((m) => {

			const texto = `
				${m.Articulo}
				${m.Usuario}
				${m.Referencia}
				${m.Observacion}
				${m.Origen}
				${m.Destino}
			`.toLowerCase();

			const coincideBusqueda =
				texto.includes(
					busqueda.toLowerCase()
				);

			const coincideTipo =
                !tipoFiltro ||
                m.Tipo === tipoFiltro;

			const coincideDesde =
				!fechaDesde ||
				m.Fecha >= fechaDesde;

			const coincideHasta =
				!fechaHasta ||
				m.Fecha <= fechaHasta;

			return (
				coincideBusqueda &&
				coincideTipo &&
				coincideDesde &&
				coincideHasta
			);
		});
    
    $: ingresos =
        movimientos.filter(
            (m) => m.Tipo === 'INGRESO'
        ).length;

    $: transferencias =
        movimientos.filter(
            (m) => m.Tipo === 'TRANSFERENCIA'
        ).length;

    $: consumos =
        movimientos.filter(
            (m) => m.Tipo === 'CONSUMO'
        ).length;

    $: devoluciones =
        movimientos.filter(
            (m) => m.Tipo === 'DEVOLUCION'
        ).length;
</script>
<div class="movimientos-container">

    <h1>Movimientos de Stock</h1>
    <div class="cards">

        <div
            class="card ingresos"
            on:click={() => tipoFiltro = 'INGRESO'}
        >
            <h3>Ingresos</h3>
            <p>{ingresos}</p>
        </div>

        <div
            class="card transferencias"
            on:click={() => tipoFiltro = 'TRANSFERENCIA'}
        >
            <h3>Transferencias</h3>
            <p>{transferencias}</p>
        </div>

        <div
            class="card consumos"
            on:click={() => tipoFiltro = 'CONSUMO'}
        >
            <h3>Consumos</h3>
            <p>{consumos}</p>
        </div>

        <div
            class="card devoluciones"
            on:click={() => tipoFiltro = 'DEVOLUCION'}
        >
            <h3>Devoluciones</h3>
            <p>{devoluciones}</p>
        </div>

    </div>
    <div class="filtros">
        <input type="date" bind:value={fechaDesde} />
        <input type="date" bind:value={fechaHasta} />
        <input
            type="text"
            placeholder="Buscar artículo, referencia, usuario..."
            bind:value={busqueda}
        />

        <select bind:value={tipoFiltro}>
            <option value="">Todos los tipos</option>
            <option>INGRESO</option>
            <option>TRANSFERENCIA</option>
            <option>CONSUMO</option>
            <option>DEVOLUCION</option>
        </select>

    </div>

    <button
        class="btn-limpiar"
        on:click={() => {
            busqueda = '';
            tipoFiltro = '';
        }}
    >
        Limpiar filtros
    </button>

    <p class="contador">
        Mostrando
        {movimientosFiltrados.length}
        movimientos
    </p>
    <div class="table-container">
        <table>

            <thead>

                <tr>
                    <th>Fecha</th>
                    <th>Tipo</th>
                    <th>Artículo</th>
                    <th>Origen</th>
                    <th>Destino</th>
                    <th>Cantidad</th>
                    <th>Usuario</th>
                    <th>Referencia</th>
                </tr>

            </thead>

            <tbody>

                {#each movimientosFiltrados as item}

                    <tr
                        on:click={() =>
                            movimientoSeleccionado = item
                        }
                    >

                        <td>
                            {formatFecha(item.Fecha)}
                        </td>

                        <td>
                            <span
                                class={`badge ${item.Tipo}`}
                            >
                                {item.Tipo}
                            </span>
                        </td>

                        <td>
                            {item.Articulo}
                        </td>

                        <td>
                            {item.Origen}
                        </td>

                        <td>
                            {item.Destino}
                        </td>

                        <td>
                            {item.Cantidad}
                        </td>

                        <td>
                            {item.Usuario}
                        </td>

                        <td>
                            {item.Referencia}
                        </td>

                    </tr>

                {/each}

            </tbody>

        </table>
    </div>
</div>
{#if movimientoSeleccionado}

<div
	class="overlay"
	on:click={() =>
		movimientoSeleccionado = null
	}
>

	<div
		class="modal"
		on:click|stopPropagation
	>

		<h2>
			Detalle Movimiento
		</h2>

		<p>
			<strong>ID:</strong>
			{movimientoSeleccionado.ID}
		</p>

		<p>
			<strong>Fecha:</strong>
			{formatFecha(
				movimientoSeleccionado.Fecha
			)}
		</p>

		<p>
			<strong>Artículo:</strong>
			{movimientoSeleccionado.Articulo}
		</p>

		<p>
			<strong>Origen:</strong>
			{movimientoSeleccionado.Origen}
		</p>

		<p>
			<strong>Destino:</strong>
			{movimientoSeleccionado.Destino}
		</p>

		<p>
			<strong>Cantidad:</strong>
			{movimientoSeleccionado.Cantidad}
		</p>

		<p>
			<strong>Tipo:</strong>
			{movimientoSeleccionado.Tipo}
		</p>

		<p>
			<strong>Usuario:</strong>
			{movimientoSeleccionado.Usuario}
		</p>

		<p>
			<strong>Referencia:</strong>
			{movimientoSeleccionado.Referencia}
		</p>

		<p>
			<strong>Observación:</strong>
			{movimientoSeleccionado.Observacion}
		</p>

		<p>
			<strong>Precio Unitario:</strong>
			{movimientoSeleccionado['Precio Unitario']}
		</p>

		<p>
			<strong>Total:</strong>
			{movimientoSeleccionado.Total}
		</p>

		<p>
			<strong>Moneda:</strong>
			{movimientoSeleccionado.Moneda}
		</p>

		{#if movimientoSeleccionado.Tipo === 'TRANSFERENCIA'
			&& movimientoSeleccionado.Referencia}

			<a
				class="btn-remito"
				href={`/remitos/${movimientoSeleccionado.Referencia}`}
				target="_blank"
			>
				📄 Ver Remito
			</a>

		{/if}

		<button
			class="cerrar-btn"
			on:click={() =>
				movimientoSeleccionado = null
			}
		>
			Cerrar
		</button>

	</div>

</div>

{/if}

<style>

    .movimientos-container {
        max-width: 1400px;
        margin: 0 auto;
        padding: 20px;
    }

    .table-container {
        width: 100%;
        overflow-x: auto;
        border-radius: 12px;
        border: 1px solid #e5e7eb;
        background: white;
        box-shadow: 0 2px 8px rgba(0,0,0,.05);
    }

    table {
        width: 100%;
        border-collapse: collapse;
        background: white;
    }

    thead {
        background: #9a9b9c;
    }

    th {
        padding: 14px;
        font-weight: 600;
        color: #374151;
        border-bottom: 1px solid #e5e7eb;
    }

    td {
        padding: 12px;
        border-bottom: 1px solid #f3f4f6;
        font-size: 12px;
    }

    tbody tr {
        cursor: pointer;
    }

    tbody tr:hover {
        background: #f9fafb;
    }

    .cards {
        display: flex;
        gap: 20px;
        margin-bottom: 24px;
        flex-wrap: wrap;
    }

    .card {
        flex: 1 1 220px;
        padding: 20px;
        border-radius: 12px;
        color: white;
        cursor: pointer;
        transition: .2s;
    }

    .card:hover {
        transform: translateY(-2px);
    }

    .card h3 {
        margin: 0;
        font-size: 16px;
    }

    .card p {
        margin: 10px 0 0;
        font-size: 28px;
        font-weight: bold;
    }

    .ingresos {
        background: #198754;
    }

    .transferencias {
        background: #2563eb;
    }

    .consumos {
        background: #fd7e14;
    }

    .devoluciones {
        background: #6f42c1;
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
        border-radius: 14px;
        width: 800px;
        max-width: 95vw;
        max-height: 90vh;
        overflow-y: auto;
        box-shadow: 0 10px 30px rgba(0,0,0,.25);
    }

    .modal h2 {
        margin-top: 0;
        margin-bottom: 20px;
        color: #111827;
    }

    .modal p {
        margin: 10px 0;
        color: #374151;
    }

    .cerrar-btn {
        margin-top: 20px;
        padding: 12px 20px;
        border: none;
        background: #2563eb;
        color: white;
        border-radius: 10px;
        cursor: pointer;
        font-weight: 600;
    }

    .btn-remito {
        display: inline-block;
        margin-top: 16px;
        padding: 12px 20px;
        background: #16a34a;
        color: white;
        text-decoration: none;
        border-radius: 10px;
        font-weight: 600;
    }

    .badge {
        padding: 6px 10px;
        border-radius: 8px;
        color: white;
        font-size: 12px;
        font-weight: 600;
    }

    .INGRESO {
        background: #198754;
    }

    .TRANSFERENCIA {
        background: #2563eb;
    }

    .CONSUMO {
        background: #fd7e14;
    }

    .DEVOLUCION {
        background: #6f42c1;
    }

    .filtros {
        display: flex;
        gap: 12px;
        margin-bottom: 16px;
        flex-wrap: wrap;
    }

    .filtros input,
    .filtros select {
        padding: 12px;
        border: 1px solid #d1d5db;
        border-radius: 10px;
        font-size: 14px;
        background: white;
        min-width: 260px;
        box-sizing: border-box;
    }

    .filtros input:focus,
    .filtros select:focus {
        outline: none;
        border-color: #2563eb;
        box-shadow: 0 0 0 3px rgba(37,99,235,.15);
    }

    .btn-limpiar {
        padding: 12px 18px;
        background: #6b7280;
        color: white;
        border: none;
        border-radius: 10px;
        cursor: pointer;
        font-weight: 600;
        margin-bottom: 12px;
        transition: .2s;
    }

    .btn-limpiar:hover {
        background: #4b5563;
    }

    .contador {
        margin: 10px 0 16px;
        color: #6b7280;
        font-size: 14px;
    }

</style>