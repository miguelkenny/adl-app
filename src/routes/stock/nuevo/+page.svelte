<script>
	import { onMount } from 'svelte';

	import { PUBLIC_API_URL } from '$env/static/public';

	const ARTICULOS_API = `${PUBLIC_API_URL}?sheet=Articulos`;
	const ALMACENES_API = `${PUBLIC_API_URL}?sheet=Almacenes`;

	let articulos = [];
	let almacenes = [];
    let guardando = false;

	let movimiento = {
		fecha: new Date().toISOString().split('T')[0],
		origen: '',
		destino: '',
		tipo: 'INGRESO',
		usuario: 'Miguel',
		referencia: '',
		observacion: ''
	};

	let items = [
        {
            articulo: '',
            busqueda: '',
            cantidad: 1
        }
    ];

	onMount(async () => {
		const [articulosRes, almacenesRes] = await Promise.all([
			fetch(ARTICULOS_API),
			fetch(ALMACENES_API)
		]);

		articulos = await articulosRes.json();
		almacenes = await almacenesRes.json();
	});

	function agregarItem() {
		items = [
            ...items,
            {
                articulo: '',
                busqueda: '',
                cantidad: 1
            }
        ];
	}

	function eliminarItem(index) {
		items = items.filter((_, i) => i !== index);
	}

	function getArticulo(nombre) {
		return articulos.find(
			(a) => a['Nombre del elemento'] === nombre
		);
	}

	function getPrecio(nombre) {
		const articulo = getArticulo(nombre);
		return Number(articulo?.['Precio']) || 0;
	}

	function getTotalItem(item) {
		return getPrecio(item.articulo) * Number(item.cantidad || 0);
	}

	$: totalGeneral = items.reduce(
		(acc, item) => acc + getTotalItem(item),
		0
	);

    async function guardarMovimiento() {
        try {
            guardando = true;
            const payload = {
                movimiento,
                items: items.map((item) => ({
                    articulo: item.articulo,
                    cantidad: Number(item.cantidad),
                    precio: getPrecio(item.articulo),
                    total: getTotalItem(item)
                }))
            };

            const response = await fetch(BASE_URL, {
                method: 'POST',
                body: JSON.stringify(payload)
            });

            const result = await response.json();

            alert(result.message);

            items = [{ articulo: '', cantidad: 1 }];

            movimiento = {
                fecha: new Date().toISOString().split('T')[0],
                origen: '',
                destino: '',
                tipo: 'INGRESO',
                usuario: 'Miguel',
                referencia: '',
                observacion: ''
            };

        } catch (error) {
            console.error(error);
            alert('Error al guardar');
        } finally {
		    guardando = false;
	    }
    }
</script>

<h1>Nuevo Movimiento</h1>

<div class="cabecera">
	<input type="date" bind:value={movimiento.fecha} />

	<select bind:value={movimiento.origen}>
		<option value="">Origen</option>
		{#each almacenes as almacen}
			<option value={almacen['Nombre']}>
				{almacen['Nombre']}
			</option>
		{/each}
	</select>

	<select bind:value={movimiento.destino}>
		<option value="">Destino</option>
		{#each almacenes as almacen}
			<option value={almacen['Nombre']}>
				{almacen['Nombre']}
			</option>
		{/each}
	</select>

	<select bind:value={movimiento.tipo}>
		<option>INGRESO</option>
		<option>TRANSFERENCIA</option>
		<option>CONSUMO</option>
		<option>AJUSTE</option>
        <option>DEVOLUCION</option>
	</select>

	<input
		type="text"
		placeholder="Referencia"
		bind:value={movimiento.referencia}
	/>

	<input
		type="text"
		placeholder="Observación"
		bind:value={movimiento.observacion}
	/>
</div>

<table>
	<thead>
		<tr>
			<th>Artículo</th>
			<th>Precio</th>
			<th>Cantidad</th>
			<th>Total</th>
			<th></th>
		</tr>
	</thead>

	<tbody>
		{#each items as item, index}
			<tr>
				<td class="articulo-cell">
                    <input
                        type="text"
                        placeholder="Buscar artículo..."
                        bind:value={item.busqueda}
                    />

                    <select bind:value={item.articulo}>
                        <option value="">Seleccionar artículo</option>

                        {#each articulos
                            .filter((articulo) => {
                                const texto = `
                                    ${articulo['Codigo Interno'] || ''}
                                    ${articulo['Codigo Proveedor'] || ''}
                                    ${articulo['Nombre del elemento'] || ''}
                                `.toLowerCase();

                                return texto.includes(
                                    (item.busqueda || '').toLowerCase()
                                );
                            })
                            .slice(0, 20) as articulo}
                            <option value={articulo['Nombre del elemento']}>
                                {articulo['Codigo Interno']} |
                                {articulo['Codigo Proveedor']} |
                                {articulo['Nombre del elemento']}
                            </option>
                        {/each}
                    </select>
                </td>

				<td>${getPrecio(item.articulo).toLocaleString()}</td>

				<td>
					<input
						type="number"
						min="1"
						bind:value={item.cantidad}
					/>
				</td>

				<td>${getTotalItem(item).toLocaleString()}</td>

				<td>
					<button on:click={() => eliminarItem(index)}>
						X
					</button>
				</td>
			</tr>
		{/each}
	</tbody>
</table>

<div class="acciones">
	<button on:click={agregarItem}>+ Agregar artículo</button>
</div>

<div class="resumen">
	<h2>Total Movimiento: ${totalGeneral.toLocaleString()}</h2>
</div>

<button
	class="guardar"
	on:click={guardarMovimiento}
	disabled={guardando}
>
	{#if guardando}
		Guardando movimiento...
	{:else}
		Guardar Movimiento
	{/if}
</button>

<style>
	h1 {
		margin-bottom: 20px;
	}

	.cabecera {
		display: grid;
		grid-template-columns: repeat(3, 1fr);
		gap: 12px;
		margin-bottom: 20px;
	}

	input,
	select,
	button {
		padding: 10px;
		border-radius: 8px;
		border: 1px solid #ccc;
	}

	table {
		width: 100%;
		border-collapse: collapse;
		margin-top: 20px;
	}

	th,
	td {
		border: 1px solid #ddd;
		padding: 10px;
	}

	.acciones {
		margin-top: 20px;
	}

	.resumen {
		margin-top: 20px;
		font-size: 1.2rem;
		font-weight: bold;
	}

	.guardar {
		margin-top: 20px;
		background: #198754;
		color: white;
		border: none;
		font-weight: bold;
		cursor: pointer;
	}

    button:disabled {
        opacity: 0.7;
        cursor: not-allowed;
    }

    .articulo-cell {
        min-width: 420px;
    }

    .articulo-cell input {
        width: 95%;
        margin-bottom: 6px;
    }

    .articulo-cell select {
        width: 100%;
    }
</style>