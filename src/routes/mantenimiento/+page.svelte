<script>
	import { onMount } from 'svelte';

	let solicitudes = [];

	const API_URL = 'TU_URL_SOLICITUDES_MANTENIMIENTO';

	onMount(async () => {
		const response = await fetch(API_URL);
		solicitudes = await response.json();
	});
</script>

<h1>Solicitudes de Mantenimiento</h1>

<table border="1">
	<thead>
		<tr>
			<th>Fecha</th>
			<th>Equipo</th>
			<th>Vehículo</th>
			<th>Urgencia</th>
			<th>Problema</th>
		</tr>
	</thead>

	<tbody>
		{#each solicitudes as item}
			<tr>
				<td>{item['Marca temporal']}</td>
				<td>{item['EQUIPMENT / EQUIPO']}</td>
				<td>{item['VEHICLE / VEHICULO']}</td>
				<td>{item['URGENCY LEVEL / NIVEL DE URGENCIA']}</td>
				<td>{item['PROBLEM / PROBLEMA / TEMA']}</td>
			</tr>
		{/each}
	</tbody>
</table>