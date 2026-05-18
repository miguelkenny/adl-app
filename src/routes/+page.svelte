<h1>ADL ERP Dashboard</h1>

<div class="stats">
	<div class="stat-card">
		<h3>--</h3>
		<p>Solicitudes pendientes</p>
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
		<h3>--</h3>
		<p>Movimientos hoy</p>
	</div>
</div>

<div class="cards">
	<a href="/compras" class="card">
		<h2>Compras</h2>
		<p>Solicitudes de materiales y seguimiento</p>
	</a>

	<a href="/mantenimiento" class="card">
		<h2>Mantenimiento</h2>
		<p>Solicitudes y seguimiento operativo</p>
	</a>

	<a href="/partes" class="card">
		<h2>Partes</h2>
		<p>Historial de trabajos realizados</p>
	</a>

	<a href="/stock" class="card">
		<h2>Stock</h2>
		<p>Artículos, movimientos y almacenes</p>
	</a>

	<a href="/stock/consumibles" class="card">
		<h2>Consumibles</h2>
		<p>Control agrupado de aceites, grasas y fluidos</p>
	</a>
</div>

<div class="ultimos">
	<h2>Últimos movimientos</h2>
	<p class="placeholder">
		Indicadores pendientes de conexión automática
	</p>
</div>

<style>
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
		box-shadow: 0 4px 12px rgba(0, 0, 0, 0.08);
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
		background: white;
		box-shadow: 0 4px 12px rgba(0, 0, 0, 0.08);
		text-decoration: none;
		color: black;
		transition: 0.2s;
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
		box-shadow: 0 4px 12px rgba(0, 0, 0, 0.08);
	}

	.placeholder {
		color: #6b7280;
		font-style: italic;
		margin-top: 10px;
	}
</style>