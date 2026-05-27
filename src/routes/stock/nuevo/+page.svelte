<script>
	import { onMount } from 'svelte';

	import { PUBLIC_API_URL } from '$env/static/public';

	const ARTICULOS_API = `${PUBLIC_API_URL}?sheet=Articulos`;
	const ALMACENES_API = `${PUBLIC_API_URL}?sheet=Almacenes`;

	let articulos = [];
	let almacenes = [];

	let loading = true;
	let guardando = false;

	let user = null;
	let almacenesPermitidos = [];

	if (typeof localStorage !== 'undefined') {
		user = JSON.parse(localStorage.getItem('user'));
	}

	let movimiento = {
		fecha: new Date().toISOString().split('T')[0],
		origen: '',
		destino: '',
		tipo: 'INGRESO',
		usuario: user?.usuario || '',
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
		try {

			const [articulosRes, almacenesRes] = await Promise.all([
				fetch(ARTICULOS_API),
				fetch(ALMACENES_API)
			]);

			articulos = await articulosRes.json();
			almacenes = await almacenesRes.json();

			almacenesPermitidos =
				user?.almacenes
					?.split(',')
					.map((a) => a.trim()) || [];

		} catch (error) {

			console.error(error);

		} finally {

			loading = false;
		}
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

		return getPrecio(item.articulo) *
			Number(item.cantidad || 0);
	}

	$: totalGeneral = items.reduce(
		(acc, item) => acc + getTotalItem(item),
		0
	);

	async function guardarMovimiento() {

		try {

			if (!movimiento.origen || !movimiento.destino) {

				alert('Debe seleccionar origen y destino');
				return;
			}

			const itemsValidos = items.filter(
				(item) =>
					item.articulo &&
					Number(item.cantidad) > 0
			);

			if (itemsValidos.length === 0) {

				alert('Debe agregar artículos válidos');
				return;
			}

			guardando = true;

			const payload = {
				movimiento,
				items: itemsValidos.map((item) => ({
					articulo: item.articulo,
					cantidad: Number(item.cantidad),
					precio: getPrecio(item.articulo),
					total: getTotalItem(item)
				}))
			};

			const response = await fetch(PUBLIC_API_URL, {
				method: 'POST',
				body: JSON.stringify(payload)
			});

			const result = await response.json();

			alert(result.message);

			items = [
				{
					articulo: '',
					busqueda: '',
					cantidad: 1
				}
			];

			movimiento = {
				fecha: new Date().toISOString().split('T')[0],
				origen: '',
				destino: '',
				tipo: 'INGRESO',
				usuario: user?.usuario || '',
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

{#if loading}

	<div class="loading">

		<div class="spinner"></div>

		<div class="loading-text">

			<h2>Cargando...</h2>

			<p>Obteniendo información de almacenes y artículos</p>

		</div>

	</div>

{:else}

	<div class="cabecera">

		<input
			type="date"
			bind:value={movimiento.fecha}
		/>

		<select bind:value={movimiento.origen}>
			<option value="">Origen</option>

			{#each almacenes.filter(
				(a) =>
					almacenesPermitidos.includes(a['Nombre']) ||
					user?.rol === 'admin'
			) as almacen}

				<option value={almacen['Nombre']}>
					{almacen['Nombre']}
				</option>

			{/each}
		</select>

		<select bind:value={movimiento.destino}>
			<option value="">Destino</option>

			{#each almacenes.filter(
				(a) =>
					almacenesPermitidos.includes(a['Nombre']) ||
					user?.rol === 'admin'
			) as almacen}

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

	<div class="table-container">

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

								<option value="">
									Seleccionar artículo
								</option>

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

						<td>
							${getPrecio(item.articulo).toLocaleString()}
						</td>

						<td>

							<input
								type="number"
								min="1"
								bind:value={item.cantidad}
							/>

						</td>

						<td>
							${getTotalItem(item).toLocaleString()}
						</td>

						<td>

							{#if items.length > 1}

								<button
									on:click={() => eliminarItem(index)}
								>
									X
								</button>

							{/if}

						</td>

					</tr>

				{/each}

			</tbody>

		</table>

	</div>

	<div class="acciones">

		<button on:click={agregarItem}>
			+ Agregar artículo
		</button>

	</div>

	<div class="resumen">

		<h2>
			Total Movimiento:
			${totalGeneral.toLocaleString()}
		</h2>

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

{/if}

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

	.table-container {
		width: 100%;
		overflow-x: auto;
	}

	table {
		width: 100%;
		border-collapse: collapse;
		margin-top: 20px;
		background: white;
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

	@media (max-width: 768px) {

		.cabecera {
			grid-template-columns: 1fr;
		}

		h1 {
			font-size: 24px;
		}
	}

	:global(main) {
		padding: 24px;
		box-sizing: border-box;
	}

	h1 {
		margin-top: 0;
	}

	table {
		border-radius: 10px;
		overflow: hidden;
		box-shadow: 0 2px 8px rgba(0,0,0,0.05);
	}

	thead {
		background: #f8fafc;
	}

	tbody tr:hover {
		background: #f9fafb;
	}

	input,
	select {
		background: white;
	}

	.guardar:hover {
		opacity: 0.92;
	}

	.loading {
		width: 100%;
		display: flex;
		justify-content: center;
		align-items: center;
		gap: 18px;
		padding: 60px 20px;
		box-sizing: border-box;
	}

	.loading-text h2 {
		margin: 0;
		font-size: 22px;
		color: #111827;
	}

	.loading-text p {
		margin: 4px 0 0;
		color: #6b7280;
		font-size: 14px;
	}

	.spinner {
		width: 42px;
		height: 42px;
		border: 4px solid #dbeafe;
		border-top: 4px solid #2563eb;
		border-radius: 50%;
		animation: spin 0.8s linear infinite;
	}

@keyframes spin {
	to {
		transform: rotate(360deg);
	}
}
	@media (max-width: 768px) {

		.cabecera {
			grid-template-columns: 1fr;
		}

		.articulo-cell {
			min-width: 300px;
		}

		:global(main) {
			padding: 14px;
		}
	}

</style>