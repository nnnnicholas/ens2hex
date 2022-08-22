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

			if (e.slice(-4) === '.eth') {
				// if it ends in .eth
				let r;
				try {
					r = await provider.resolveName(e);
				} catch (error) {
					alert(e.toString() + ' is not a valid name');
					console.error('Promise rejected!', error);
				}
				r === null
					? alert(
							e.toString() +
								" does not exist or it's Resolver has not been configured to point to an Ethereum address."
					  )
					: b.push(r);
				console.log(r);
			} else {
				// if it doesn't end in .eth
				let r;
				try {
					r = await ethers.utils.getAddress(e);
				} catch (error) {
					alert(e.toString() + ' is not a valid .eth or hex address.');
					console.error('Promise rejected!', error);
				}
				r === null ? alert(e.toString() + ' is not a valid .eth or hex address.') : b.push(r);
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
	<h1
		class="text-4xl tracking-tight font-bold text-gray-800 sm:text-5xl sm:tracking-tight md:text-6xl md:tracking-tight lg:text-5xl lg:tracking-tight xl:text-6xl xl:tracking-tight pb-2 "
	>
		ens2hex
	</h1>
	<code class="tracking-tight text-[#666666] font-semibold italic sm:w-1/2 text-xs sm:text-sm"
		>Convert a CSV list of ENS and hex (0x) format addresses to checksummed hex addresses.</code
	>

	<p class="pt-8 sm:pt-10 block text-sm sm:text-lg sm:pl-1  font-medium text-gray-700">
		Addresses to convert
	</p>

	<p
		class="sm:pl-1 text-xs hidden sm:flex sm:text-sm pb-1"
		style="color: grey; font-style: italic; margin-top: 0; padding-top: 0"
	>
		Comma separated (CSV) list of addresses (with or without spaces)
	</p>
	<p
		class="sm:pl-1 text-xs sm:hidden pb-1"
		style="color: grey; font-style: italic; margin-top: 0; padding-top: 0"
	>
		Comma-separated list of addresses (with or w/o spaces)
	</p>

	<textarea
		id="input"
		class="shadow-sm p-3 sm:p-4 placeholder:italic rounded-2xl focus:ring-indigo-500 focus:border-indigo-500 block w-full text-xs sm:text-sm border border-gray-300 "
		bind:value={input}
		on:keypress={onKeyPress}
		placeholder="ie. nnnnicholas.eth, 0x2638bf07baa4a246af902ad19d3d14e0dff0ce97, vitalik.eth, ..."
	/>

	<div id="actions" class="flex gap-5 pt-4 justify-between sm:justify-end">
		<div class="flex items-center gap-1">
			<label for="deduplication" class="text-sm font-bold">Deduplicate?</label>
			<!-- <input type="checkbox" id="deduplication" bind:checked={deduplication} /> -->
			<input
				id="deduplication"
				bind:checked={deduplication}
				aria-describedby="comments-description"
				name="deduplication"
				type="checkbox"
				class="focus:ring-indigo-[#63A591] h-4 w-4 text-[#63A591] border-gray-300 rounded"
			/>
		</div>
		<div>
			<button
				type="button"
				on:click={update}
				class="inline-flex items-center px-4 py-2  border border-transparent text-sm font-medium rounded-md shadow-sm text-white  bg-[#63A591] hover:bg-opacity-80 focus:outline-none focus:ring-2 focus:ring-offset-2 transition focus:ring-[#63A591]"
			>
				Convert
			</button>
		</div>
	</div>

	<p class="pt-10 block text-sm sm:text-lg sm:pl-1 pb-1 font-medium text-gray-700">Results</p>
	<p
		class="sm:pl-1 text-xs hidden sm:flex sm:text-sm pb-1"
		style="color: grey; font-style: italic; margin-top: 0; padding-top: 0"
	>
		Click convert to see your results below.
	</p>
	<textarea
		class="shadow-sm p-3 sm:p-4 placeholder:italic rounded-2xl focus:ring-indigo-500 focus:border-indigo-500 block w-full text-xs sm:text-sm border border-gray-300 mb-4"
		id="output"
		bind:value={output}
		readonly
	/>
	<div id="spacer" />
	<div id="footer" class="flex gap-2 transition">
		<code href="https://twitter.com/nnnnicholas">@nnnnicholas</code>
		•
		<code href="https://github.com/nnnnicholas/ens2hex">github</code>
	</div>
</body>

<style>
	body {
		display: flex;
		flex-direction: column;
		min-height: 100vh;
		margin: 0 40px;
		background: #f5f5f5;
		padding-top: 15px;
		margin: auto;
		max-width: 975px;
	}
	@media (max-width: 575px) {
		body {
			margin: 0 12px;
			padding-top: 12px;
		}
	}
	textarea {
		height: 10em;
	}
	p {
		margin-top: 0;
		/* color: #161616; */
	}
	#spacer {
		flex-grow: 1;
	}
	#footer {
		align-self: center;
		padding-bottom: 40px;
	}
	#footer code:hover {
		cursor: pointer;
		color: #63a591;
		text-decoration: underline;
		transition-property: color, background-color, border-color, text-decoration-color, fill, stroke,
			opacity, box-shadow, transform, filter, backdrop-filter;
		transition-timing-function: cubic-bezier(0.4, 0, 0.2, 1);
		transition-duration: 150ms;
	}
</style>
