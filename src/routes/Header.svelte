<script>
	import { resolve } from '$app/paths';
	import { page } from '$app/state';
	let menuOpen = false;
</script>

<header>
	<div class="logo">
		<a href={resolve('/')}>
			<h1>ADL ERP</h1>
			<span>Control Operativo y Logístico</span>
		</a>
	</div>
	<button
		class="menu-btn"
		on:click={() => (menuOpen = !menuOpen)}
	>
		☰
	</button>
	<nav class:open={menuOpen}>
		<ul>
			<li aria-current={page.url.pathname === '/' ? 'page' : undefined}>
				<a href={resolve('/')}
				on:click={() => (menuOpen = false)}
				>Dashboard</a>
			</li>

			<li aria-current={page.url.pathname.startsWith('/compras') ? 'page' : undefined}>
				<a href={resolve('/compras')}
				on:click={() => (menuOpen = false)}
				>Compras</a>
			</li>

			<li
				class="dropdown"
				aria-current={page.url.pathname.startsWith('/stock') ? 'page' : undefined}
			>
				<a href={resolve('/stock')}
				on:click={() => (menuOpen = false)}>Stock ▾</a>

				<ul class="submenu">
					<li>
						<a href={resolve('/stock')}>Stock General</a>
					</li>

					<li>
						<a href={resolve('/stock/consumibles')}>Consumibles</a>
					</li>

					<li>
						<a href={resolve('/stock/nuevo')}>Nuevo Movimiento</a>
					</li>
				</ul>
			</li>

			<li
				class="dropdown"
				aria-current={page.url.pathname.startsWith('/mantenimiento') ? 'page' : undefined}
			>
				<a href={resolve('/mantenimiento')}
				on:click={() => (menuOpen = false)}>Mantenimiento ▾</a>

				<ul class="submenu">
					<li>
						<a href={resolve('/mantenimiento')}>
							Solicitudes
						</a>
					</li>

					<li>
						<a href={resolve('/partes/vehiculos')}>
							Partes Vehículos
						</a>
					</li>

					<li>
						<a href={resolve('/partes/equipos')}>
							Partes Equipos
						</a>
					</li>
				</ul>
			</li>
			<li aria-current={page.url.pathname.startsWith('/operaciones') ? 'page' : undefined}>
				<a href={resolve('/operaciones')}
				on:click={() => (menuOpen = false)}>
					Operaciones
				</a>
			</li>
		</ul>
	</nav>
</header>

<style>
	header {
		display: flex;
		justify-content: space-between;
		align-items: center;
		padding: 18px 32px;
		background: #111827;
		color: white;
		box-shadow: 0 2px 10px rgba(0, 0, 0, 0.15);
		position: sticky;
		top: 0;
		z-index: 100;
	}

	.logo a {
		text-decoration: none;
		color: white;
		display: flex;
		flex-direction: column;
	}

	.logo h1 {
		margin: 0;
		font-size: 24px;
		font-weight: 700;
	}

	.logo span {
		font-size: 12px;
		color: #9ca3af;
		margin-top: 2px;
	}

	nav ul {
		display: flex;
		gap: 16px;
		list-style: none;
		margin: 0;
		padding: 0;
		align-items: center;
	}

	nav li {
		position: relative;
	}

	nav a {
		color: white;
		text-decoration: none;
		padding: 10px 14px;
		border-radius: 8px;
		font-weight: 600;
		font-size: 14px;
		transition: all 0.2s ease;
	}

	nav a:hover {
		background: rgba(255, 255, 255, 0.08);
	}

	li[aria-current='page'] > a {
		background: #2563eb;
	}

	.dropdown {
		position: relative;
	}

	.submenu {
		display: none;
		position: absolute;
		top: 100%;
		left: 0;
		background: white;
		min-width: 180px;
		border-radius: 10px;
		box-shadow: 0 8px 18px rgba(0, 0, 0, 0.12);
		padding: 8px 0;
		list-style: none;
		margin-top: 8px;
		z-index: 1000;
	}

	.submenu li {
		width: 100%;
	}

	.submenu a {
		display: block;
		padding: 12px 16px;
		color: #111827;
		text-decoration: none;
		font-size: 14px;
		font-weight: 500;
		border-radius: 0;
	}

	.submenu a:hover {
		color: #2563eb;
	}

	.dropdown:hover .submenu {
		display: block;
	}
	
	.menu-btn {
		display: none;
		background: transparent;
		border: none;
		color: white;
		font-size: 28px;
		cursor: pointer;
	}

	@media (max-width: 768px) {

	header {
		flex-wrap: wrap;
		padding: 16px;
	}

	.menu-btn {
		display: block;
	}

	nav {
		width: 100%;
		display: none;
	}

	nav.open {
		display: block;
	}

	nav ul {
		flex-direction: column;
		align-items: stretch;
		gap: 8px;
		margin-top: 16px;
	}

	nav a {
		display: block;
		width: 100%;
		box-sizing: border-box;
	}

	.dropdown .submenu {
		position: static;
		display: block;
		background: #1f2937;
		box-shadow: none;
		margin-top: 6px;
		border-radius: 8px;
		padding: 6px;
	}

	.submenu a {
		color: white;
		padding-left: 24px;
	}

	.submenu a:hover {
		background: rgba(255,255,255,0.08);
		color: white;
	}

	.logo h1 {
		font-size: 20px;
	}

	.logo span {
		font-size: 11px;
	}
}
</style>