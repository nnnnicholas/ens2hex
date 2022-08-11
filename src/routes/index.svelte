<script lang="ts">
	import {
		accountChainId,
		connected,
		connectMetamask,
		connectWalletConnect,
		disconnect,
		walletAddress
	} from '$lib/stores/provider';
	import { balanceOnBlock } from '$lib/stores/state';
	import { ethers } from 'ethers';
	import { formatEther } from 'ethers/lib/utils';

	const provider = ethers.getDefaultProvider('mainnet');

	let input = '';
	let output='';
	let b=[];
	async function convert() {
		// output = ''
		// b = [];
		let i = JSON.parse(JSON.stringify(input)); // duplicate object
		i = i.replace(/\s+/g, ''); // remove spaces
		let a = i.split(',');
		a.forEach(async (e) => {
			e = e.toLowerCase(); // convert to lowercase
			if (e.slice(0, 2) == '0x') { // if hex address
				let r;
				try {
					r = await ethers.utils.getAddress(e);
					// e = r;
					b.push(r);
				} catch (error) {
					console.error('Promise rejected!', error);
				}
			} else {
				let r; 
				try {
					r = await provider.resolveName(e);
					// e=r;
					b.push(r);
				} catch(error) {
					console.error('Promise rejected!', error);
				}
				console.log(r);
			}
		});
		// console.log(b);
		// output = b.toString();
	}
	async function update(){
		await convert();
		output = b.toString();
	}
</script>

<body>
	<h1>ENS2WalletConverter</h1>
	<p>
		Paste a CSV list of ENS and/or hex (0x) format addresses. Press Convert to convert them to all
		hex format addresses.
	</p>
	<h3>Addresses to convert</h3>
	<p style="color: grey; font-style: italic; margin-top: 0; padding-top: 0">
		CSV values (with or without spaces)
	</p>
	<textarea
		id="input"
		bind:value={input}
		placeholder="nnnnicholas.eth, 0x2638bf07baa4a246af902ad19d3d14e0dff0ce97, vitalik.eth, ..."
	/>
	<input id="button" type="button" value="Convert" on:click={update} />
	<h3>Results</h3>
	<textarea id="output" bind:value={output}
	readonly />
</body>

<!-- <main class="flex flex-col p-10 w-screen h-screen items-center gap-5 font-serif">
	  <h1 class="text-5xl text-black font-bold">Svelte Ethers Template</h1>
  
	  <section class="flex flex-col p-10 items-start gap-5 border-2 rounded-xl">
		  <h1 class="text-2xl text-black font-bold">Wallet and Provider</h1>
		  <h2 class="text-xl text-black font-semibold">$connected: {$connected}</h2>
		  <h2 class="text-xl text-black font-semibold">$walletAddress: {$walletAddress}</h2>
		  <h2 class="text-xl text-black font-semibold">
			  $accountChainId: {JSON.stringify($accountChainId)}
		  </h2>
		  <button
			  on:click={connectMetamask}
			  class="p-3 rounded-xl text-xl bg-black text-white font-semibold hover:scale-[1.05] transition transition-200"
			  >Connect via MetaMask</button
		  >
		  <button
			  on:click={connectWalletConnect}
			  class="p-3 rounded-xl text-xl bg-black text-white font-semibold hover:scale-[1.05] transition transition-200"
			  >Connect via WalletConnect</button
		  >
		  <button
			  on:click={disconnect}
			  class="p-3 rounded-xl text-xl bg-black text-white font-semibold hover:scale-[1.05] transition transition-200"
			  >Disconnect</button
		  >
	  </section>
	  <section class="flex flex-col p-10 items-start gap-5 border-2 rounded-xl">
		  <h1 class="text-2xl text-black font-bold">Sync contract state on block, event or interval</h1>
		  <p class="text-md text-black font-semibold">
			  Set In '$lib/globalState.ts', will load as long as a store is used from '$lib/stores/state.ts'
			  or you import the globalState in the root of your project with 'import "$lib/globalState";'
		  </p>
		  <h2 class="text-xl text-black font-semibold">
			  $balanceOnBlock: {formatEther($balanceOnBlock)} ETH
		  </h2>
	  </section> 
	
</main>
-->
<style>
	body {
		display: flex;
		flex-direction: column;
	}
	textarea {
		height: 10em;
	}
	p {
		margin-top: 0;
	}
</style>
