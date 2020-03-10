<script>
	import { onMount } from "svelte"
	import Map from './Map.svelte';
	import { csv, json } from 'd3-fetch';
	let active = 'Confirmed';
	let data = null;
	let world = null;
	let isInitial = false;

	const handleClick = (type) => () => {
		active = type
	};

	onMount(async function () {
		[world, data] = await Promise.all([
			json("https://cdn.jsdelivr.net/npm/world-atlas@2/countries-110m.json"),
			csv("https://docs.google.com/spreadsheets/d/e/2PACX-1vRjixGgr6I18VAxY6Ybv26aOeBdfD-NqHU01nnjBR-rpPPs0Jhtgvmaclyo98kG4XXWKiIUyY5vbA3Q/pub?gid=3549213&single=true&output=csv")
		]);
		isInitial = true
	})
</script>

<main>
	{#if isInitial}
		<Map active={active} data={data} world={world} />
		<div class="label-box">
			<label class:active={active === 'Confirmed'} on:click={handleClick('Confirmed')}><span style="background: #B00020"></span>總確診</label>
			<span>=</span>
			<label class:active={active === 'Treatment'} on:click={handleClick('Treatment')}><span style="background: #dc8c50"></span>治療中</label>
			<span>+</span>
			<label class:active={active === 'Recovered'} on:click={handleClick('Recovered')}><span style="background: #90EE02"></span>復原</label>
			<span>+</span>
			<label class:active={active === 'Deaths'} on:click={handleClick('Deaths')}><span style="background: #000"></span>死亡</label>
		</div>
	{/if}
</main>

<style>
	main {
		text-align: center;
		margin: 0 auto;
		overflow: hidden;
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
	}

	.label-box label {
		border-radius: 16px;
		height: 32px;
		line-height: 32px;
		padding: 0 12px;

		margin: 0 8px;
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
