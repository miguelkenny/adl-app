<script>
	const formularios = [
		{
			titulo: 'Parte Diario Hincado Ullum Alfa 2026',
			url: 'https://forms.gle/ka6dn4NYi4neNRqw8'
		},
        {
            titulo: 'Parte Diario Perforación Casposo',
            url: 'https://forms.gle/cAKphUJAneNmtx5w6'
        },
		{
			titulo: 'Registro Movimiento de Móviles',
			url: 'https://forms.gle/ZCgTcmwmna3qYffX7'
		},
		{
			titulo: 'Solicitud de Materiales',
			url: 'https://forms.gle/qLVeDKgEYW6x9bmq8'
		},
		{
			titulo: 'Solicitud / Recepción EPP',
			url: 'https://forms.gle/S9HD1bYH4zpq3qj67'
		},
		{
			titulo: 'Parte Mantenimiento Móviles',
			url: 'https://forms.gle/fnB4UCgXF4X9ZAdf9'
		},
		{
			titulo: 'Parte Mantenimiento Equipos',
			url: 'https://forms.gle/6SDJBaK9BQ8bPjKH7'
		},
		{
			titulo: 'Solicitud de Mantenimiento',
			url: 'https://forms.gle/Qq3rBrRrN1F7tqam6'
		},
        {
            titulo: 'V0624 Registro Movimiento de Moviles',
            url: 'https://forms.gle/aaXNGDaWGXJ5p8Dd8'
        }
	];

	const manuales = [
		{
			titulo: 'Manual Hincadora Turchi',
			url: 'https://drive.google.com/file/d/1HsrPpO_dgPJy-PU0ieJSSKJxbSLuWN9V/view?usp=sharing'
		}
	];
</script>

<h1>Accesos Operativos</h1>

<h2>Formularios Operativos</h2>

<div class="grid">
	{#each formularios as item}
		<a
			class="card formulario"
			href={item.url}
			target="_blank"
			rel="noopener noreferrer"
		>
			<div class="badge azul">Formulario</div>

			<h3>{item.titulo}</h3>

			<span>Abrir formulario</span>
		</a>
	{/each}
</div>

<h2 class="manuales-title">
	Manuales y Documentación
</h2>

<div class="grid">
	{#each manuales as item}
		<a
			class="card manual"
			href={item.url}
			target="_blank"
			rel="noopener noreferrer"
		>
			<div class="badge gris">Manual</div>

			<h3>{item.titulo}</h3>

			<span>Abrir documento</span>
		</a>
	{/each}
</div>

<style>
	h1 {
		margin-bottom: 30px;
	}

	h2 {
		margin-top: 30px;
		margin-bottom: 16px;
		color: #111827;
	}

	.manuales-title {
		margin-top: 40px;
	}

	.grid {
		display: grid;
		grid-template-columns: repeat(auto-fit, minmax(260px, 1fr));
		gap: 20px;
	}

	.card {
		background: white;
		border-radius: 14px;
		padding: 20px;
		text-decoration: none;
		color: #111827;
		border: 1px solid #ddd;
		transition: all 0.2s ease;
		display: flex;
		flex-direction: column;
		gap: 14px;
	}

	.card:hover {
		transform: translateY(-4px);
		box-shadow: 0 8px 20px rgba(0,0,0,0.10);
	}

	.formulario:hover {
		border-color: #2563eb;
	}

	.manual:hover {
		border-color: #6b7280;
	}

	.badge {
		width: fit-content;
		padding: 6px 10px;
		border-radius: 8px;
		font-size: 12px;
		font-weight: bold;
		color: white;
	}

	.azul {
		background: #2563eb;
	}

	.gris {
		background: #6b7280;
	}

	h3 {
		margin: 0;
		font-size: 18px;
		line-height: 1.4;
	}

	span {
		margin-top: auto;
		font-weight: bold;
	}
</style>