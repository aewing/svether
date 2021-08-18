<script>
	import '../app.postcss';
	import { ethers } from 'ethers';
	import { onMount } from 'svelte';

	let hasMetamask = false;
	let connected = false;
	let address;
	let balance;
	let chain;
	let blockNumber;
	onMount(async () => {
		if (window.ethereum) {
			hasMetamask = true;
			chain = new ethers.providers.Web3Provider(window.ethereum);
			blockNumber = await chain.getBlockNumber();
			connected = true;
			await requestAccounts();
			window.ethereum.on('accountsChanged', onAccountsChanged);
		}
	});

	async function onAccountsChanged(accounts) {
		address = accounts[0];
		balance = await chain.getBalance(address);
	}

	async function requestAccounts() {
		if (hasMetamask) {
			const accounts = await window.ethereum.request({ method: 'eth_requestAccounts' });
			if (accounts) {
				await onAccountsChanged(accounts);
			}
		}
	}
</script>

<header>
	<a id="brand" href="/">svether</a>
	<nav>
		<a href="/">dashboard</a>
		<a href="/transaction">transaction</a>
		<a href="/contract">contract</a>
	</nav>
</header>

<div id="chainstatus">
	<section>
		<label for="chain_status">Chain Status:</label>
		<div class="pill" id="chain_status">
			{#if !hasMetamask}
				<div class="dot bg-red-500" />
				No Metamask
			{:else if !connected}
				<div class="dot bg-yellow-500" />
				Not connected
			{:else}
				<div class="dot bg-green-500" />
				Connected
			{/if}
		</div>
	</section>
	<section>
		<label for="chain_status">Block:</label>
		<div class="pill" id="chain_status">
			{#if !blockNumber}
				<div class="dot bg-yellow-500" />
				Not connected
			{:else}
				<div class="dot bg-green-500" />
				{blockNumber}
			{/if}
		</div>
	</section>
	{#if !address}
		<button on:click={requestAccounts}>Connect</button>
	{:else}
		<section>
			<label for="wallet_address">Wallet Address:</label>
			<div class="pill" id="wallet_address">
				{#if !address}
					<div class="dot bg-yellow-500" />
					Not connected
				{:else}
					<div class="dot bg-green-500" />
					{address}
				{/if}
			</div>
		</section>
		<section>
			<label for="wallet_balance">Balance:</label>
			<div class="pill" id="wallet_balance">
				{#if balance === undefined}
					<div class="dot bg-yellow-500" />
					Not connected
				{:else}
					<div class="dot bg-green-500" />
					{balance}
				{/if}
			</div>
		</section>
	{/if}
</div>

<main>
	<slot />
</main>

<style>
	:root {
		@apply w-full h-full bg-gray-900 text-white block;
		font-family: 'VictorMono NF', 'Operator Mono', 'DejaVu Sans Mono', 'Droid Sans Mono', monospace;
		font-size: 18px;
	}

	header {
		@apply bg-gray-900 flex justify-between items-center px-4 py-2;
	}
	header a#brand {
		@apply text-2xl text-green-300;
	}
	header nav {
		@apply flex;
	}
	header nav a {
		@apply px-4 py-1 text-purple-400 border-b-2 border-transparent text-lg transition-all;
	}
	header nav a:hover {
		@apply text-green-300 border-b-2 border-green-200;
	}

	button {
		@apply px-4 py-1 bg-purple-800 rounded text-lg transition-all ml-4 border-none text-white;
	}

	#chainstatus {
		@apply p-2 flex justify-between;
	}

	#chainstatus section {
		@apply px-2 py-1 inline-flex justify-center items-center;
	}
	#chainstatus section label {
		@apply text-lg mr-2;
	}

	.pill {
		@apply bg-gray-800 px-2 py-1 rounded inline-flex justify-center items-center;
	}

	.dot {
		@apply rounded-full w-3 h-3 mr-2;
	}

	main {
		@apply p-4;
	}
</style>
