<script>
	import { onMount } from 'svelte';
	import { PUBLIC_API_URL } from '$env/static/public';

	let loading = true;
	let remitos = [];
    let busqueda = '';

	onMount(async () => {

		try {

			const response = await fetch(
				`${PUBLIC_API_URL}?sheet=Movimientos`
			);

			const movimientos =
				await response.json();

			const agrupados = {};

			movimientos
				.filter(
					(m) =>
						m.Tipo === 'TRANSFERENCIA' &&
						m.Referencia
				)
				.forEach((mov) => {

					if (!agrupados[mov.Referencia]) {

						agrupados[mov.Referencia] = {

							referencia:
								mov.Referencia,

							fecha:
								mov.Fecha,

							origen:
								mov.Origen,

							destino:
								mov.Destino,

							items: 0
						};
					}

					agrupados[
						mov.Referencia
					].items++;
				});

			remitos =
				Object.values(
					agrupados
				).sort(
					(a, b) =>
						new Date(b.fecha) -
						new Date(a.fecha)
				);

		} catch (error) {

			console.error(error);

		} finally {

			loading = false;
		}
	});

    function formatFecha(fecha) {

        if (!fecha) return '';

        return new Date(fecha)
            .toLocaleDateString('es-AR');
    }

</script>

<div class="page-container">
    <h1>Remitos de Obras</h1>

    <input
        class="search-box"
        type="text"
        placeholder="Buscar remito, obra o almacén..."
        bind:value={busqueda}
    />
    {#if loading}

        <p>Cargando...</p>

    {:else}
        <p class="info">
            Mostrando
            {remitos.filter((r) => {

                const texto = `
                    ${r.referencia}
                    ${r.origen}
                    ${r.destino}
                `.toLowerCase();

                return texto.includes(
                    busqueda.toLowerCase()
                );

            }).length}
            remitos
        </p>
        <div class="table-container">
            <table>

                <thead>

                    <tr>
                        <th>Fecha</th>
                        <th>Remito</th>
                        <th>Origen</th>
                        <th>Destino</th>
                        <th>Items</th>
                        <th></th>
                    </tr>

                </thead>

                <tbody>

                    {#each remitos.filter((r) => {

                        const texto = `
                            ${r.referencia}
                            ${r.origen}
                            ${r.destino}
                        `.toLowerCase();

                        return texto.includes(
                            busqueda.toLowerCase()
                        );

                    }) as remito}

                        <tr>

                            <td>
                                {formatFecha(remito.fecha)}
                            </td>

                            <td>
                                <span class="remito-id">
                                    {remito.referencia}
                                </span>
                            </td>

                            <td>
                                {remito.origen}
                            </td>

                            <td>
                                {remito.destino}
                            </td>

                            <td>
                                {remito.items}
                            </td>

                            <td>

                                <a
                                    class="btn-ver"
                                    href={`/remitos/${remito.referencia}`}
                                >
                                    Ver
                                </a>

                            </td>

                        </tr>

                    {/each}

                </tbody>

            </table>
        </div>

    {/if}
</div>
<style>

	:global(body) {
		background: #f3f4f6;
	}

	.page-container {
		padding: 24px;
		max-width: 1400px;
		margin: 0 auto;
	}

	h1 {
		margin-bottom: 20px;
		color: #111827;
		font-size: 28px;
		font-weight: 700;
	}

	.search-box {
		width: 100%;
		max-width: 450px;
		padding: 12px 16px;
		border: 1px solid #d1d5db;
		border-radius: 10px;
		font-size: 14px;
		background: white;
		margin-bottom: 16px;
		box-sizing: border-box;
	}

	.search-box:focus {
		outline: none;
		border-color: #2563eb;
		box-shadow: 0 0 0 3px rgba(37,99,235,.15);
	}

	.info {
		margin-bottom: 20px;
		color: #6b7280;
		font-size: 14px;
	}

	.table-container {
		background: white;
		border-radius: 14px;
		box-shadow: 0 4px 12px rgba(0,0,0,.08);
		overflow: hidden;
	}

	table {
		width: 100%;
		border-collapse: collapse;
	}

	th {
		background: #f3f4f6;
		color: #374151;
		font-weight: 600;
		font-size: 14px;
		padding: 14px;
		text-align: left;
		border-bottom: 1px solid #e5e7eb;
	}

	td {
		padding: 14px;
		border-bottom: 1px solid #f1f5f9;
		font-size: 14px;
		color: #111827;
	}

	tbody tr:hover {
		background: #f9fafb;
	}

	.remito-id {
		font-weight: 700;
		color: #2563eb;
	}

	.btn-ver {
		display: inline-block;
		padding: 8px 14px;
		background: #2563eb;
		color: white;
		text-decoration: none;
		border-radius: 8px;
		font-size: 13px;
		font-weight: 600;
		transition: .2s;
	}

	.btn-ver:hover {
		background: #1d4ed8;
		transform: translateY(-1px);
	}

	.loading {
		background: white;
		padding: 30px;
		border-radius: 14px;
		text-align: center;
		box-shadow: 0 4px 12px rgba(0,0,0,.08);
		color: #6b7280;
	}

	@media (max-width: 768px) {

		.page-container {
			padding: 14px;
		}

		.table-container {
			overflow-x: auto;
		}

		table {
			min-width: 800px;
		}

		h1 {
			font-size: 24px;
		}
	}

</style>