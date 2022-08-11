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

	const RPC_HOST = `https://cloudflare-eth.com/`;
	const provider = new ethers.providers.JsonRpcProvider(RPC_HOST);

	let input = '';
	let output = '';
	let deduplication = false;
	let b = [];
	async function convert() {
		let i = JSON.parse(JSON.stringify(input)); // duplicate object
		i = i.replace(/\s+/g, ''); // remove spaces
		let a = i.split(',');
		for (let e of a) {
			e = e.toLowerCase(); // convert to lowercase
			if (e.slice(0, 2) == '0x') {
				// if hex address
				let r;
				try {
					r = await ethers.utils.getAddress(e);
				} catch (error) {
					alert(e.toString() + ' is not a valid hex address');
					console.error('Promise rejected!', error);
				}
				r === null ? alert(e.toString() + ' is not a valid hex address') : b.push(r);
				console.log(r);
			} else {
				let r;
				try {
					r = await provider.resolveName(e);
				} catch (error) {
					alert(e.toString() + ' is not a valid name');
					console.error('Promise rejected!', error);
				}
				r === null ? alert(e.toString() + ' is not a valid name') : b.push(r);
				console.log(r);
			}
		}
	}

	 function dedupe(array) {
		return [...new Set(array)];
	}

	async function emptyOutput() {
		output = '';
		b = [];
	}
	async function update() {
		await emptyOutput();
		convert()
			.then(() => {
				deduplication === true ? (b = dedupe(b)) : '';
			})
			.then(() => {
				// b = b.reverse();
				output = b.toString();
			});
	}

	const onKeyPress = (e) => {
		if (e.charCode === 13) {
			e.preventDefault();
			update();
		}
	};
</script>

<body>
	<h1>ens2hex</h1>
	<p>Convert a CSV list of ENS and hex (0x) format addresses to checksummed hex addresses.</p>
	<h3>Addresses to convert</h3>
	<p style="color: grey; font-style: italic; margin-top: 0; padding-top: 0">
		CSV list of addresses (with or without spaces)
	</p>
	<textarea
		id="input"
		bind:value={input}
		on:keypress={onKeyPress}
		placeholder="nnnnicholas.eth, 0x2638bf07baa4a246af902ad19d3d14e0dff0ce97, vitalik.eth, ..."
	/>
	<div id = "actions">
		<div>
			<label for="deduplication">Deduplicate</label>
			<input type="checkbox" id="deduplication" bind:checked={deduplication} />
		</div>
		<input id="button" type="button" value="Convert" on:click={update} />
	</div>
	<h3>Results</h3>
	<textarea id="output" bind:value={output} readonly />
	<div id="spacer" />
	<div id="footer">
		<a href="https://github.com/nnnnicholas/ens2hex">github</a>
	</div>
</body>

<style>
	body {
		display: flex;
		flex-direction: column;
		min-height: 100vh;
		margin: 0 40px;
	}
	textarea {
		height: 10em;
	}
	p {
		margin-top: 0;
	}
	#spacer {
		flex-grow: 1;
	}
	#footer {
		align-self: end;
		padding-bottom: 40px;
	}
	#actions{
		display: flex;
		flex-direction: row;
		justify-content: space-between;
	}
	#button{
		flex-grow:1;
	}
</style>
