<script>
	export let mensaje = 'Cargando información...';
	export let subtitulo = 'Aguarde un instante';
</script>

<div class="loader">
	<h2>{mensaje}</h2>
	<p>{subtitulo}</p>
</div>

<style>
	.loader {
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: center;
		padding: 80px;
		text-align: center;
		font-family: Arial;
		color: #555;
	}

	.loader h2 {
		margin-bottom: 10px;
	}

	.loader::after {
		content: '';
		width: 40px;
		height: 40px;
		margin-top: 20px;
		border: 4px solid #ddd;
		border-top: 4px solid #0d6efd;
		border-radius: 50%;
		animation: spin 1s linear infinite;
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