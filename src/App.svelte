<script>
	import { onMount } from 'svelte';
	import Map from './Map.svelte';
	import { csv, json } from 'd3-fetch';
	import { format } from 'd3-format';
	import Table from './Table.svelte';

	let active = 'Confirmed';
	let data = null;
	let world = null;
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

	onMount(async function () {
		[world, data] = await Promise.all([
			json('https://cdn.jsdelivr.net/npm/world-atlas@2/countries-110m.json'),
			csv('https://docs.google.com/spreadsheets/d/e/2PACX-1vRjixGgr6I18VAxY6Ybv26aOeBdfD-NqHU01nnjBR-rpPPs0Jhtgvmaclyo98kG4XXWKiIUyY5vbA3Q/pub?gid=3549213&single=true&output=csv')
		]);

		sum.Confirmed = formatNumber(sumByType(data, 'Confirmed'));
		sum.Treatment = formatNumber(sumByType(data, 'Treatment'));
		sum.Recovered = formatNumber(sumByType(data, 'Recovered'));
		sum.Deaths = formatNumber(sumByType(data, 'Deaths'));

		isInitial = true;
	});
</script>

<main>
	{#if isInitial}
		<div class="cut">
			<Map active={active} data={data} world={world} />
			<div class="label-box">
				<label class:active={active === 'Confirmed'} on:click={handleClick('Confirmed')}><span style="background: #B00020"></span>總確診({sum.Confirmed})</label>
				<span>=</span>
				<label class:active={active === 'Treatment'} on:click={handleClick('Treatment')}><span style="background: #dc8c50"></span>治療中({sum.Treatment})</label>
				<span>+</span>
				<label class:active={active === 'Recovered'} on:click={handleClick('Recovered')}><span style="background: #90EE02"></span>復原({sum.Recovered})</label>
				<span>+</span>
				<label class:active={active === 'Deaths'} on:click={handleClick('Deaths')}><span style="background: #000"></span>死亡({sum.Deaths})</label>
			</div>
		</div>
		<Table data={data} />
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
</style>
