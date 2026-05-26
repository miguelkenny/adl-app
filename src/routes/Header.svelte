<script>
	import { resolve } from '$app/paths';
	import { page } from '$app/state';
	import { goto } from '$app/navigation';
	import { onMount } from 'svelte';

	let menuOpen = false;

	let usuario = null;

	onMount(() => {

		const user =
			localStorage.getItem('user');

		if (user) {
			usuario = JSON.parse(user);
		}
	});

	function logout() {

		localStorage.removeItem('user');

		goto('/login');
	}
</script>

<header>
	<div class="logo">
		<a href={resolve('/')}>
			<h1>ADL / ANDESITA ERP</h1>
			<span>Control Operativo y Logístico</span>
		</a>
	</div>

	<button
		type="button"
		class="menu-btn"
		onclick={() => (menuOpen = !menuOpen)}
	>
		☰
	</button>

	<nav class={menuOpen ? 'open' : ''}>
		<ul>
			<li aria-current={page.url.pathname === '/' ? 'page' : undefined}>
				<a
					href={resolve('/')}
					onclick={() => (menuOpen = false)}
				>
					Dashboard
				</a>
			</li>

			<li aria-current={page.url.pathname.startsWith('/compras') ? 'page' : undefined}>
				<a
					href={resolve('/compras')}
					onclick={() => (menuOpen = false)}
				>
					Compras
				</a>
			</li>

			<li
				class="dropdown"
				aria-current={page.url.pathname.startsWith('/stock') ? 'page' : undefined}
			>
				<a href={resolve('/stock')}>
					Stock ▾
				</a>

				<ul class="submenu">
					<li>
						<a
							href={resolve('/stock')}
							onclick={() => (menuOpen = false)}
						>
							Stock General
						</a>
					</li>

					<li>
						<a
							href={resolve('/stock/consumibles')}
							onclick={() => (menuOpen = false)}
						>
							Consumibles
						</a>
					</li>

					<li>
						<a
							href={resolve('/stock/nuevo')}
							onclick={() => (menuOpen = false)}
						>
							Nuevo Movimiento
						</a>
					</li>
				</ul>
			</li>

			<li
				class="dropdown"
				aria-current={page.url.pathname.startsWith('/mantenimiento') ? 'page' : undefined}
			>
				<a href={resolve('/mantenimiento')}>
					Mantenimiento ▾
				</a>

				<ul class="submenu">
					<li>
						<a
							href={resolve('/mantenimiento')}
							onclick={() => (menuOpen = false)}
						>
							Solicitudes de Mantenimiento
						</a>
					</li>

					<li>
						<a
							href={resolve('/partes/vehiculos')}
							onclick={() => (menuOpen = false)}
						>
							Partes Vehículos
						</a>
					</li>

					<li>
						<a
							href={resolve('/partes/equipos')}
							onclick={() => (menuOpen = false)}
						>
							Partes Equipos
						</a>
					</li>
				</ul>
			</li>

			<li aria-current={page.url.pathname.startsWith('/operaciones') ? 'page' : undefined}>
				<a
					href={resolve('/formularios')}
					onclick={() => (menuOpen = false)}
				>
					Formularios
				</a>
			</li>
			<li class="user-info">

				<div class="user-badge">

					<span class="user-name">
						{usuario?.usuario}
					</span>

					<span class="user-role">
						{usuario?.rol}
					</span>

				</div>

			</li>

			<li>
				<button
					class="logout-btn"
					onclick={logout}
				>
					Salir
				</button>
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

nav {
	transition: all 0.2s ease;
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
	display: block;
}

nav a:hover {
	background: rgba(255, 255, 255, 0.08);
}

li[aria-current='page'] > a {
	background: #2563eb;
}

/* =========================
	DROPDOWN DESKTOP
========================= */

.dropdown {
	position: relative;
}

.submenu {
	display: none;
	position: absolute;
	top: 100%;
	left: 0;
	background: white;
	min-width: 220px;
	border-radius: 10px;
	box-shadow: 0 8px 18px rgba(0, 0, 0, 0.12);
	padding: 8px 0;
	list-style: none;
	margin-top: 0; /* IMPORTANTE */
	z-index: 1000;
}

.dropdown:hover .submenu {
	display: block;
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
	background: #f3f4f6;
	color: #2563eb;
}

/* =========================
	HAMBURGUESA
========================= */

.menu-btn {
	display: none;
	background: transparent;
	border: none;
	color: white;
	font-size: 28px;
	cursor: pointer;
}

.logout-btn {
	background: #dc3545;
	color: white;
	border: none;
	padding: 10px 14px;
	border-radius: 8px;
	font-weight: 600;
	font-size: 14px;
	cursor: pointer;
	transition: 0.2s;
}

.logout-btn:hover {
	background: #bb2d3b;
}

.user-info {
	display: flex;
	align-items: center;
}

.user-badge {
	display: flex;
	flex-direction: column;
	align-items: flex-end;
	padding: 6px 10px;
	background: rgba(255,255,255,0.08);
	border-radius: 8px;
}

.user-name {
	font-size: 14px;
	font-weight: 700;
	color: white;
	line-height: 1.1;
}

.user-role {
	font-size: 11px;
	color: #9ca3af;
	text-transform: uppercase;
}

/* =========================
	MOBILE
========================= */

@media (max-width: 768px) {

	header {
		flex-wrap: wrap;
		padding: 16px;
	}

	.logo {
		flex: 1;
	}

	.logo h1 {
		font-size: 20px;
	}

	.logo span {
		font-size: 11px;
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
		width: 100%;
		box-sizing: border-box;
	}

	/* SUBMENU MOVIL */
	.dropdown .submenu {
		position: static;
		display: block;
		background: #1f2937;
		box-shadow: none;
		margin-top: 4px;
		border-radius: 8px;
		padding: 6px;
		min-width: 100%;
	}

	.submenu a {
		color: white;
		padding-left: 24px;
		border-radius: 8px;
	}

	.submenu a:hover {
		background: rgba(255,255,255,0.08);
		color: white;
	}

	.logout-btn {
		width: 100%;
		text-align: left;
	}

	.user-badge {
		align-items: flex-start;
		width: 100%;
	}
}
</style>