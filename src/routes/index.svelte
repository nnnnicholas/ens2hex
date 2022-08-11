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
	import { AbiCoder, formatEther } from 'ethers/lib/utils';

	const RPC_HOST = `https://cloudflare-eth.com/`;
	const provider = new ethers.providers.JsonRpcProvider(RPC_HOST);

	let input = '';
	let output = '';
	let deduplication = false;
	let receivable = false;
	let b = [];
	async function convert() {
		let i = JSON.parse(JSON.stringify(input)); // duplicate object
		i = i.replace(/\s+/g, ''); // remove spaces
		let a = i.split(',');
		for (let e of a) {
			e = e.toLowerCase(); // convert to lowercase
			if (e.slice(0, 2) == '0x' && e.length == 42) {
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

	async function isContract(address) {
		// returns true if passed address is a contract
		if (address.slice(0, 2) != '0x' || address.length != 42) {
			// check if input string is an address
			console.error('isContract(): Malformed address input');
		} else {
			try {
				const code = await provider.getCode(address);
				if (code !== '0x') return true; // is a contract
			} catch (error) {
				console.error(error);
			}
			return false; // is not a contract
		}
	}

	async function erc721receiver(array) {
		let problems = []; // addresses corresponding to contracts that do not support IERC721Receiver
		for (let e of array) {
			// Check if it's a contract
			let isItAContract;
			try {
				isItAContract = await isContract(e);
			} catch (error) {
				console.error(error);
			}
			if (isItAContract === true) {
				try {
					//Check if it supports ERC721Receiver
					const abi = [
						' function supportsInterface(bytes4 interfaceID) external view returns (bool)'
					];
					let contract = new ethers.Contract(e, abi, provider);
					contract.supportsInterface('onERC721Received(address,address,uint256,bytes)') ===
					'0x150b7a02'
						? '' // does support
						: problems.push(e); // does NOT support
				} catch (error) {
					console.error(error);
				}
			}
		}
		return problems; // Return array of contract addresses that do not implement ERC721Receiver
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
			.then(async () => {
				if (receivable === true) {
					let probs = await erc721receiver(b);
					probs.length != 0
						? 	alert(
								'The following addresses correspond to contracts that do not implement ERC721Receivable and will fail a safeTransferFrom of an NFT.\n\n' +
									probs.toString()
						  )
						: '';
				}
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
	<div id="actions">
		<div>
			<input type="checkbox" id="deduplication" bind:checked={deduplication} />
			<label for="deduplication">Deduplicate</label>
			<br>
			<input type="checkbox" id="receivable" bind:checked={receivable} />
			<label for="receivable">Supports ERC721Receiver</label>
		</div>
		<input id="button" type="button" value="Convert" on:click={update} />
	</div>
	<h3>Results</h3>
	<textarea id="output" bind:value={output} readonly />
	<div id="spacer" />
	<div id="footer">
		<a href="https://twitter.com/nnnnicholas">@nnnnicholas</a> |
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
	#actions {
		display: flex;
		flex-direction: row;
		justify-content: space-between;
	}
	#button {
		/* flex-grow: 1; */
		width: 80px;
		height: 44px;
		background-color: rgb(120, 120, 208);
		color:white;
	}
</style>
