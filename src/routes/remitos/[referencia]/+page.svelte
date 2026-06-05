<script>
	import { page } from '$app/state';
	import { PUBLIC_API_URL } from '$env/static/public';
	import { onMount } from 'svelte';

	let movimientos = [];
	let articulos = [];
	let remito = null;

	const localidades = {
		Casposo: 'Villa Corral, Calingasta',
		Gualcamayo: 'Jáchal',
		Veladero: 'Iglesia'
	};

	function getArticulo(nombre) {
		return articulos.find(
			(a) => a['Nombre del elemento'] === nombre
		);
	}

	function formatFecha(fecha) {
		if (!fecha) return '';

		return new Date(fecha).toLocaleDateString('es-AR');
	}

	onMount(async () => {

		const referencia = page.params.referencia;

		const [movResponse, artResponse] =
			await Promise.all([
				fetch(
					`${PUBLIC_API_URL}?sheet=Movimientos`
				),
				fetch(
					`${PUBLIC_API_URL}?sheet=Articulos`
				)
			]);

		const data = await movResponse.json();

		articulos = await artResponse.json();

		movimientos = data.filter(
			(item) =>
				item.Tipo === 'TRANSFERENCIA' &&
				item.Referencia === referencia
		);

		if (movimientos.length > 0) {
			remito = movimientos[0];
		}
	});
</script>

{#if remito}

<div class="toolbar no-print">

	<div>
		<h1>
			Remito {remito.Referencia}
		</h1>

		<p>
			Transferencia de materiales
		</p>
	</div>

	<button
		class="print-btn"
		onclick={() => window.print()}
	>
		🖨 Imprimir
	</button>

</div>

<div class="hoja">

	<!-- FECHA -->

	<div class="fecha">
		{formatFecha(remito.Fecha)}
	</div>

	<!-- DESTINO -->

	<div class="titulo-destino">
		Destino:
	</div>

	<div class="valor-destino">
		{remito.Destino}
	</div>

	<!-- LOCALIDAD -->

	<div class="titulo-localidad">
		Localidad:
	</div>

	<div class="valor-localidad">
		{localidades[remito.Destino] || ''}
	</div>

	<!-- DETALLE -->

	{#each movimientos as item, i}

		{@const articulo = getArticulo(item.Articulo)}

		<div
			class="fila-remito"
			style={`top:${325 + (i * 17)}px`}
		>

			<div class="cant">
				{item.Cantidad}
			</div>

			<div class="um">
				{articulo?.['Unidad Medida'] || ''}
			</div>

			<div class="codigo">
				{articulo?.['Codigo Proveedor'] || ''}
			</div>

			<div class="detalle">
				{item.Articulo}
			</div>

		</div>

	{/each}

</div>

{/if}

<style>

    * {
        box-sizing: border-box;
    }

    :global(body) {
        background: #f3f4f6;
    }

    h1 {
        margin: 0;
        color: #111827;
        font-size: 20px;
    }

    .toolbar p {
        margin: 4px 0 0;
        color: #6b7280;
        font-size: 14px;
    }

	.toolbar {
        max-width: 1000px;
        margin: 20px auto;
        padding: 24px;
        background: white;
        border-radius: 14px;
        box-shadow: 0 4px 12px rgba(0,0,0,.08);

        display: flex;
        justify-content: space-between;
        align-items: center;
    }

	.print-btn {
        background: #2563eb;
        color: white;
        border: none;
        margin-left: 20px;
        padding: 12px 20px;
        border-radius: 10px;
        font-weight: 600;
        cursor: pointer;
        transition: .2s;
    }

    .print-btn:hover {
        background: #1d4ed8;
        transform: translateY(-1px);
    }

	.hoja {
        position: relative;
        width: 210mm;
        height: 297mm;
        margin: auto;
        background: white;
        font-family: Arial, sans-serif;
        font-size: 11px;
    }

	/* ==========================
	   FECHA (G2)
	========================== */

	.fecha {
		position: absolute;
		top: 105px;
		left: 490px;
		width: 140px;
		text-align: center;
	}

	/* ==========================
	   DESTINO (B4 / D4)
	========================== */

	.titulo-destino {
		position: absolute;
		top: 235px;
		left: 50px;
		font-weight: normal;
	}

	.valor-destino {
		position: absolute;
		top: 235px;
		left: 130px;
	}

	/* ==========================
	   LOCALIDAD (B5 / D5)
	========================== */

	.titulo-localidad {
		position: absolute;
		top: 255px;
		left: 50px;
		font-weight: normal;
	}

	.valor-localidad {
		position: absolute;
		top: 255px;
		left: 130px;
	}

	/* ==========================
	   FILAS DEL REMITO
	========================== */

	.fila-remito {
		position: absolute;
		width: 100%;
		height: 17px;
		font-size: 11px;
	}

	/* B */

	.cant {
		position: absolute;
		left: 50px;
		width: 50px;
		text-align: center;
	}

	/* C */

	.um {
		position: absolute;
		left: 100px;
		width: 31px;
		text-align: center;
	}

	/* D */

	.codigo {
		position: absolute;
		left: 155px;
		width: 85px;
	}

	/* E */

	.detalle {
		position: absolute;
		left: 280px;
		width: 420px;
	}

	@media print {

        @page {
            size: A4 portrait;
            margin: 0;
        }

        html,
        body {
            width: 210mm;
            height: 297mm;
            margin: 0;
            padding: 0;
            overflow: hidden;
        }

        .no-print {
            display: none !important;
        }

        .hoja {
            width: 210mm;
            height: 297mm;
            margin: 0;
            padding: 0;
            box-shadow: none;
            overflow: hidden;
            page-break-after: avoid;
            page-break-before: avoid;
            page-break-inside: avoid;
        }
    }

</style>