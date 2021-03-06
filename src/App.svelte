<script>
	import { onMount } from 'svelte';
	import Map from './Map.svelte';
	import { csv, json } from 'd3-fetch';
	import { format } from 'd3-format';
	import archieML from 'archieml';
	import Table from './Table.svelte';
	import Loader from './Loader.svelte';
	import Papa from 'papaparse';

	let active = 'Confirmed';
	let map = null;
	let world = null;
	let table = null;
	let text = null;
	let isInitial = false;

	const sum = {};

	const handleClick = (type) => () => {
		active = type;
	};

	const sumByType = (data, type) => {
		return data.reduce((acc, row) => {
			return parseInt(row[type]) + acc;
		}, 0);
	};

	const formatNumber = format('.3s');

	const fetchSheet = ({ id, name }) => {
		const url = `https://sheets.googleapis.com/v4/spreadsheets/${id}/values/${name}?key=${process.env.GOOGLE_API_KEY}`;
		return fetch(url).then(response => response.json().then(result => {
			return Promise.resolve(Papa.parse(Papa.unparse(result.values), { header: true }).data)
		}))
	};

	const fetchDoc = ({ id }) => {
		const url = `https://www.googleapis.com/drive/v3/files/${id}/export?key=${process.env.GOOGLE_API_KEY}&mimeType=text/plain`
		return fetch(url).then(response => response.text().then((text) => {
			return archieML.load(text)
		}))
	}


	onMount(async function () {
		[world, map, table, text] = await Promise.all([
			json('https://cdn.jsdelivr.net/npm/world-atlas@2/countries-110m.json'),
			fetchSheet({ id: '17wMAD2zzIP3Arst0naOQ425ibV2YSVcDjyxFHh5bikM', name: 'Export-map' }),
			fetchSheet({ id: '17wMAD2zzIP3Arst0naOQ425ibV2YSVcDjyxFHh5bikM', name: 'Export-table' }),
			fetchDoc({ id: '10PahzeKS_O9IaGLsG2vEswhVScHIaZ9uEUDbSMll_Sg' })
		]);

		sum.Confirmed = formatNumber(sumByType(table, 'Confirmed'));
		sum.Treatment = formatNumber(sumByType(table, 'Treatment'));
		sum.Recovered = formatNumber(sumByType(table, 'Recovered'));
		sum.Deaths = formatNumber(sumByType(table, 'Deaths'));

		isInitial = true;
	});
</script>

<main>
	{#if isInitial}
		{#each text.layout as cut}
			{#if cut === 'map'}
				<div class="cut">
					<Map active={active} data={map} world={world} text={text} />
					<div class="label-box">
						<label class:active={active === 'Confirmed'} on:click={handleClick('Confirmed')}><span style="background: #B00020"></span>{text.lang.confirmed}({sum.Confirmed})</label>
						<span>=</span>
						<label class:active={active === 'Treatment'} on:click={handleClick('Treatment')}><span style="background: #dc8c50"></span>{text.lang.treatment}({sum.Treatment})</label>
						<span>+</span>
						<label class:active={active === 'Recovered'} on:click={handleClick('Recovered')}><span style="background: #90EE02"></span>{text.lang.recovered}({sum.Recovered})</label>
						<span>+</span>
						<label class:active={active === 'Deaths'} on:click={handleClick('Deaths')}><span style="background: #000"></span>{text.lang.deaths}({sum.Deaths})</label>
					</div>
				</div>
			{:else if cut === 'table'}
				<Table data={table} text={text} />
			{:else if cut === 'footer'}
				<div class="copyright">
					{#each text.footers as item}
						<span>{item.front}<a href="{item.link}">{item.text}</a></span>
					{/each}
				</div>
			{/if}
		{/each}
	{:else}
		<Loader />
	{/if}
</main>

<style>
	main {
		text-align: center;
		margin: 0 auto;
	}

	.cut {
		display: flex;
		justify-content: center;
		align-items: center;
		flex-direction: column;
		height: 100vh;
	}

	h1 {
		color: #ff3e00;
		text-transform: uppercase;
		font-size: 4em;
		font-weight: 100;
	}

	.label-box {
		display: flex;
		color: #fff;
		margin-top: 20px;
		align-items: center;
		letter-spacing: 2px;
		flex-wrap: wrap;
		justify-content: center;
		max-width: 80vw;
	}

	.label-box label {
		border-radius: 16px;
		height: 32px;
		line-height: 32px;
		padding: 0 12px;
		margin: 6px 8px;
		background: #505050;
		opacity: 0.3;
		display: flex;
		align-items: center;
		font-weight: bold;
	}
	.label-box label span{
		width: 18px;
		height: 18px;
		border-radius: 50%;
		display: inline-block;
		margin-right: 6px;
	}

	.label-box label.active {
		opacity: 1;
	}

	.copyright {
		padding: 40px 0;
		color: rgba(255, 255, 255, 0.6);
	}

	.copyright a {
		text-decoration: underline;
		color: rgba(255, 255, 255, 0.6);
	}

	.copyright span:not(:first-child) {
		margin-left: 10px;
	}
</style>
