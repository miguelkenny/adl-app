<script>
	import { onMount } from 'svelte';
	import { PUBLIC_API_URL, PUBLIC_API_URL_COMPRAS } from '$env/static/public';

	let movimientos = [];
	let compras = [];
	let usuarios = [];
	let usuario = null;

	let movimientosHoy = 0;
	let comprasPendientes = 0;
	let loading = true;

	const MOVIMIENTOS_API =
		`${PUBLIC_API_URL}?sheet=Movimientos`;
	
	const COMPRAS_API =
		`${PUBLIC_API_URL_COMPRAS}?sheet=Respuestas`;

	const USUARIOS_API =
		`${PUBLIC_API_URL}?sheet=Usuarios`;

	onMount(async () => {
		try {
			const user =
			localStorage.getItem('user');

			if (user) {
				usuario = JSON.parse(user);
			}
			const [movRes, comprasRes, usuariosRes] = await Promise.all([
				fetch(MOVIMIENTOS_API),
				fetch(COMPRAS_API),
				fetch(USUARIOS_API)
			]);

			movimientos = await movRes.json();
			compras = await comprasRes.json();
			usuarios = await usuariosRes.json();
			console.log(usuarios);
			
			calcularIndicadores();

		} catch (error) {
			console.error(error);
		}
		finally {
			loading = false;
		}
	});

	function calcularIndicadores() {

		const hoy = new Date().toISOString().split('T')[0];

		movimientosHoy = movimientos.filter((item) => {

			const fechaMovimiento =
				item.Fecha?.split('T')[0];

			return fechaMovimiento === hoy;

		}).length;

		comprasPendientes = compras.filter((item) => {

			const estado =
				item['ESTADO']?.trim();

			return (
				estado === 'Pendiente' ||
				estado === 'Presupuestando' ||
				estado === 'En proceso compra' ||
				estado === 'Comprado esperando llegada'
			);

		}).length;
	}
</script>
<div class="dashboard-container">
	<h1>ADL / ANDESITA - ERP Dashboard</h1>

	<div class="welcome">
		<h2>
			Hola, {usuario?.usuario}
		</h2>

		<p>
			Resumen operativo del sistema
		</p>
	</div>

	<h2 class="section-title">
		Indicadores
	</h2>

	<div class="stats">
		<div class="stat-card">
			<h3>
				{#if loading}
					<div class="spinner"></div>
				{:else}
					{comprasPendientes}
				{/if}
			</h3>
			<p>Solicitudes De Compras Pendientes</p>
		</div>

		<div class="stat-card warning">
			<h3>--</h3>
			<p>Stock crítico</p>
		</div>

		<div class="stat-card danger">
			<h3>--</h3>
			<p>Consumibles críticos</p>
		</div>

		<div class="stat-card success">
			<h3>
				{#if loading}
					<div class="spinner"></div>
				{:else}
					{movimientosHoy}
				{/if}
			</h3>
			<p>Movimientos hoy</p>
		</div>
	</div>

	<h2 class="section-title">
		Módulos
	</h2>

	<div class="cards">

		{#if usuario?.rol === 'admin' || usuario?.rol === 'supervisor'}
			<a href="/compras" class="card compras">
				<h2>Compras</h2>
				<p>Solicitudes de materiales y seguimiento</p>
			</a>
		{/if}

		<a href="/mantenimiento" class="card mantenimiento">
			<h2>Mantenimiento</h2>
			<p>Solicitudes y seguimiento operativo</p>
		</a>

		<a href="/partes" class="card partes">
			<h2>Partes</h2>
			<p>Historial de trabajos realizados</p>
		</a>

		{#if usuario?.rol === 'admin' || usuario?.rol === 'supervisor'}
			<a href="/stock" class="card stock">
				<h2>Stock</h2>
				<p>Artículos, movimientos y almacenes</p>
			</a>
		{/if}

		<a href="/stock/consumibles" class="card consumibles">
			<h2>Consumibles</h2>
			<p>Control agrupado de aceites, grasas y fluidos</p>
		</a>
		{#if usuario?.rol === 'admin' || usuario?.rol === 'supervisor'}
			<a href="/informes/envios-obras" class="card informes">
				<h2>Informes</h2>
				<p>Informes de Envíos a Obras</p>
			</a>

			<a href="/remitos/ver" class="card remitos">
				<h2>Remitos</h2>
				<p>Ver remitos historicos</p>
			</a>
		{/if}
		
	</div>

	<div class="ultimos">
		<div class="ultimos-header">
			<h2>Últimos Movimientos Registrados</h2>

			{#if !loading}
				<span>{movimientos.length} registros</span>
			{/if}
		</div>

		{#if loading}

			<div class="skeleton-container">
				<div class="loading-line"></div>
				<div class="loading-line"></div>
				<div class="loading-line"></div>
				<div class="loading-line"></div>
				<div class="loading-line"></div>
			</div>

		{:else}

			<div class="ultimos-table">
				<table>
					<thead>
						<tr>
							<th>Fecha</th>
							<th>Artículo</th>
							<th>Tipo</th>
							<th>Cantidad</th>
							<th>Usuario</th>
						</tr>
					</thead>

					<tbody>
						{#each movimientos.slice(-5).reverse() as mov}
							<tr>
								<td>{mov.Fecha?.split('T')[0]}</td>

								<td class="articulo">
									{mov.Articulo}
								</td>

								<td>
									<span
										class={`tipo ${mov.Tipo?.toLowerCase()}`}
									>
										{mov.Tipo}
									</span>
								</td>

								<td>{mov.Cantidad}</td>

								<td>{mov.Usuario}</td>
							</tr>
						{/each}
					</tbody>
				</table>
			</div>

		{/if}
	</div>
</div>
<style>
	.dashboard-container {
		padding: 0px 14px;
	}
	h1 {
		font-size: 32px;
		margin-bottom: 24px;
		color: #111827;
	}

	.stats {
		display: grid;
		grid-template-columns: repeat(auto-fit, minmax(220px, 1fr));
		gap: 20px;
		margin-bottom: 30px;
	}

	.stat-card {
		background: white;
		padding: 24px;
		border-radius: 14px;
		box-shadow: 0 4px 12px rgba(0,0,0,.08);
		border-left: 6px solid #2563eb;
	}

	.stat-card h3 {
		font-size: 34px;
		margin: 0;
		color: #111827;
	}

	.stat-card p {
		margin-top: 8px;
		color: #6b7280;
		font-size: 14px;
	}

	.warning h3 {
		color: #d97706;
	}

	.danger h3 {
		color: #dc2626;
	}

	.success h3 {
		color: #16a34a;
	}

	.warning {
		border-left-color: #f59e0b;
	}

	.danger {
		border-left-color: #ef4444;
	}

	.success {
		border-left-color: #22c55e;
	}

	.cards {
		display: grid;
		grid-template-columns: repeat(auto-fit, minmax(260px, 1fr));
		gap: 20px;
		margin-bottom: 30px;
	}

	.card {
		display: block;
		padding: 24px;
		border-radius: 14px;

		background: linear-gradient(
			135deg,
			#ffffff,
			#f8fafc
		);

		border: 1px solid #e5e7eb;

		box-shadow:
			0 4px 12px rgba(0,0,0,.05);

		text-decoration: none;
		color: inherit;

		transition: all .2s ease;
	}

	.card:hover {
		transform: translateY(-4px);
		box-shadow: 0 8px 18px rgba(0, 0, 0, 0.12);
	}

	.card h2 {
		margin-top: 0;
		color: #111827;
	}

	.card p {
		color: #6b7280;
		font-size: 14px;
	}

	.ultimos {
		background: white;
		padding: 24px;
		border-radius: 14px;
		box-shadow: 0 4px 12px rgba(0,0,0,.08);
	}

	.ultimos-header {
		display: flex;
		justify-content: space-between;
		align-items: center;
		margin-bottom: 18px;
	}

	.ultimos-header h2 {
		margin: 0;
		font-size: 20px;
		color: #111827;
	}

	.ultimos-header span {
		font-size: 13px;
		color: #6b7280;
	}

	.ultimos-table {
		overflow-x: auto;
	}

	.ultimos table {
		width: 100%;
		border-collapse: collapse;
	}

	.ultimos th {
		background: #f8fafc;
		padding: 12px;
		text-align: left;
		font-size: 13px;
		font-weight: 600;
		color: #374151;
		border-bottom: 2px solid #e5e7eb;
	}

	.ultimos td {
		padding: 12px;
		border-bottom: 1px solid #e5e7eb;
		font-size: 14px;
	}

	.ultimos tbody tr:hover {
		background: #f9fafb;
	}

	.tipo {
		padding: 4px 10px;
		border-radius: 999px;
		font-size: 12px;
		font-weight: 600;
		display: inline-block;
	}

	.tipo.ingreso {
		background: #dcfce7;
		color: #166534;
	}

	.tipo.consumo {
		background: #fee2e2;
		color: #991b1b;
	}

	.tipo.ajuste {
		background: #dbeafe;
		color: #1d4ed8;
	}

	.tipo.transferencia {
		background: #ede9fe;
		color: #6d28d9;
	}

	.articulo {
		font-weight: 500;
		color: #111827;
	}

	.placeholder {
		color: #6b7280;
		font-style: italic;
		margin-top: 10px;
	}

	.skeleton-container {
		display: flex;
		flex-direction: column;
		gap: 12px;
	}

	.loading-line {
		height: 18px;
		border-radius: 8px;
		background: #c4c5c7;
		animation: pulse 1.5s infinite;
	}

	.loading-line:nth-child(1) {
		width: 100%;
	}

	.loading-line:nth-child(2) {
		width: 90%;
	}

	.loading-line:nth-child(3) {
		width: 95%;
	}

	.loading-line:nth-child(4) {
		width: 85%;
	}

	.loading-line:nth-child(5) {
		width: 92%;
	}
	.spinner {
		width: 28px;
		height: 28px;
		border: 3px solid #e5e7eb;
		border-top: 3px solid #2563eb;
		border-radius: 50%;
		animation: spin 0.8s linear infinite;
	}

	.welcome {
		background: linear-gradient(
			135deg,
			#2563eb,
			#1d4ed8
		);

		color: white;

		padding: 24px;

		border-radius: 14px;

		margin-bottom: 24px;
	}

	.welcome h2 {
		margin: 0;
	}

	.welcome p {
		margin-top: 6px;
		opacity: .9;
	}

	.section-title {
		margin: 28px 0 16px;
		font-size: 20px;
		font-weight: 700;
		color: #374151;
	}

	.compras {
		border-top: 4px solid #2563eb;
	}

	.mantenimiento {
		border-top: 4px solid #f59e0b;
	}

	.partes {
		border-top: 4px solid #8b5cf6;
	}

	.stock {
		border-top: 4px solid #22c55e;
	}

	.consumibles {
		border-top: 4px solid #14b8a6;
	}

	.informes {
		border-top: 4px solid #ec4899;
	}

	.remitos {
		border-top: 4px solid #4c136d;
	}

	@keyframes pulse {
		0% {
			opacity: 0.5;
		}

		50% {
			opacity: 1;
		}

		100% {
			opacity: 0.5;
		}
	}

	@keyframes spin {
		from {
			transform: rotate(0deg);
		}
		to {
			transform: rotate(360deg);
		}
	}
</style>