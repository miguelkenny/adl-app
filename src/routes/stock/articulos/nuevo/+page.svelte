<script>
	let guardando = false;

	const API_URL =
		'https://script.google.com/macros/s/AKfycbyOL2jeaK79ebykD_zfx1Eqg1yFoBFvZQPoqnL4MAlG--CPtPiBg93J6AcCpRTFbw6WbQ/exec';

	let articulo = {
		codigoProveedor: '',
		nombre: '',
		contenedor: '',
		tipo: '',
		marca: '',
		unidadMedida: '',
		precio: '',
		consumible: '',
		stockMinimo: '',
		notas: ''
	};

	async function guardarArticulo() {
		try {
			guardando = true;

			const response = await fetch(API_URL, {
				method: 'POST',
				body: JSON.stringify({
					action: 'crearArticulo',
					articulo
				})
			});

			const result = await response.json();

			alert(result.message || 'Artículo creado correctamente');

			articulo = {
				codigoProveedor: '',
				nombre: '',
				contenedor: '',
				tipo: '',
				marca: '',
				unidadMedida: '',
				precio: '',
				consumible: '',
				stockMinimo: '',
				notas: ''
			};
		} catch (error) {
			console.error(error);
			alert('Error al guardar artículo');
		} finally {
			guardando = false;
		}
	}
</script>

<h1>Nuevo Artículo</h1>

<div class="formulario">
	<input
		type="text"
		placeholder="Código Proveedor"
		bind:value={articulo.codigoProveedor}
	/>

	<input
		type="text"
		placeholder="Nombre del elemento"
		bind:value={articulo.nombre}
	/>

	<input
		type="text"
		placeholder="Contenedor"
		bind:value={articulo.contenedor}
	/>

	<select bind:value={articulo.tipo}>
		<option value="">Tipo</option>
		<option>Lubricantes</option>
		<option>Herramientas</option>
		<option>Seguridad</option>
		<option>Hincadora</option>
		<option>Vehiculos</option>
		<option>Instalaciones</option>
		<option>Repuestos</option>
		<option>Filtros</option>
		<option>Perforación</option>
		<option>EPP</option>
		<option>Mangueras</option>
		<option>Transporte</option>
		<option>Insumos</option>
		<option>Varios</option>
	</select>

	<input
		type="text"
		placeholder="Marca"
		bind:value={articulo.marca}
	/>

	<select bind:value={articulo.unidadMedida}>
		<option value="">Unidad de medida</option>
		<option>UN</option>
		<option>LT</option>
		<option>KG</option>
		<option>MT</option>
		<option>JGO</option>
	</select>

	<input
		type="number"
		placeholder="Precio"
		bind:value={articulo.precio}
	/>

	<select bind:value={articulo.consumible}>
		<option value="">Consumible</option>
		<option>NO</option>
		<option>ACEITE HIDRAULICO</option>
		<option>ACEITE DE COMPRESOR</option>
		<option>ACEITE MARTILLO</option>
		<option>ACEITE DE MOTOR</option>
		<option>GRASA ROSCAS DE BARRA</option>
		<option>GRASA MULTI-PROPOSITO</option>
	</select>

	<input
		type="number"
		placeholder="Stock mínimo"
		bind:value={articulo.stockMinimo}
	/>

	<textarea
		placeholder="Notas"
		bind:value={articulo.notas}
	></textarea>
</div>

<button
	class="guardar-btn"
	on:click={guardarArticulo}
	disabled={guardando}
>
	{#if guardando}
		Guardando artículo...
	{:else}
		Guardar Artículo
	{/if}
</button>

<style>
	h1 {
		margin-bottom: 24px;
		font-family: Arial, sans-serif;
	}

	.formulario {
		display: grid;
		grid-template-columns: repeat(2, 1fr);
		gap: 14px;
		max-width: 900px;
	}

	input,
	select,
	textarea {
		padding: 12px;
		border: 1px solid #ccc;
		border-radius: 8px;
		font-size: 14px;
		font-family: Arial, sans-serif;
	}

	textarea {
		grid-column: span 2;
		min-height: 120px;
		resize: vertical;
	}

	.guardar-btn {
		margin-top: 24px;
		padding: 14px 20px;
		background: #198754;
		color: white;
		border: none;
		border-radius: 8px;
		font-weight: bold;
		cursor: pointer;
		font-size: 15px;
	}

	.guardar-btn:hover {
		opacity: 0.9;
	}

	.guardar-btn:disabled {
		opacity: 0.7;
		cursor: not-allowed;
	}
</style>