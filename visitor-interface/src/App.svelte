<script>
	import { ethers } from "ethers";
	import QRCode from "qrcode";
	import { onMount } from "svelte";
	import { getTexts } from "./helpers.js";

	let walletInfos = [];
	let amountOfWallets = 99;
	let counter = 0;
	let generationCompleted = false;
	let resultOfCreateRandom;
	let texts = {};
	let ready = false;
	let showQRCodesAlso = false;

	onMount(async () => {
		texts = getTexts();
		ready = true;
	});

	// async function generateOne() {
	// 	let walletInfo = {};
	// 	resultOfCreateRandom = ethers.Wallet.createRandom();

	// 	const walletFromChainCode = new ethers.Wallet(
	// 		resultOfCreateRandom.chainCode,
	// 	);

	// 	walletInfo.address = walletFromChainCode.address;
	// 	walletInfo.privateKey = walletFromChainCode.privateKey;
	// 	walletInfo.mnemonic = resultOfCreateRandom.mnemonic.phrase;
	// 	walletInfo.canvasIDAddress = `canvasAddress${counter}`;
	// 	walletInfo.canvasIDPrivateKey = "canvasPrivateKey" + counter;

	// 	return walletInfo;
	// }

	function generateOne() {
		let walletInfo = {};
		const resultOfCreateRandom = ethers.Wallet.createRandom();
		const checker = ethers.Wallet.fromPhrase(resultOfCreateRandom.mnemonic.phrase);
        // this.logger.warning(checker)
		walletInfo.address = resultOfCreateRandom.address;
		walletInfo.privateKey = resultOfCreateRandom.privateKey;
		walletInfo.mnemonic = resultOfCreateRandom.mnemonic.phrase;
		walletInfo.canvasIDAddress = `canvasAddress${counter}`;
		walletInfo.canvasIDPrivateKey = "canvasPrivateKey" + counter;

        if (walletInfo.address !== checker.address) throw new Error("what")
        if (walletInfo.privateKey !== checker.privateKey) throw new Error("the")
        if (walletInfo.mnemonic !== checker.mnemonic.phrase) throw new Error("fuck")
		return walletInfo;
	}
	
	function prepareQRCodes(data) {
		for (const walletInfo of data) {
			QRCode.toCanvas(
				document.getElementById(walletInfo.canvasIDAddress),
				walletInfo.address,
				{ errorCorrectionLevel: "M", version: 4 },
				function (error) {
					if (error) console.error(error);
					console.log("qr code addresssuccess!");
				},
			);
			QRCode.toCanvas(
				document.getElementById(walletInfo.canvasIDPrivateKey),
				walletInfo.privateKey,
				{ errorCorrectionLevel: "M", version: 5 },
				function (error) {
					if (error) console.error(error);
					console.log("qr code private key success!");
				},
			);
		}
	}
	async function generateAll() {
		walletInfos = [];

		while (counter < amountOfWallets) {
			counter++;
			let walletInfo = await generateOne();
			walletInfos.push(walletInfo);
		}
		generationCompleted = true;

		if (showQRCodesAlso) {
			setTimeout(() => {
				prepareQRCodes(walletInfos);
			}, 1 * 200);
		}
	}
</script>

<main>
	{#if ready}
		{#if !generationCompleted}
			<div class="noprint">
				<h3>Paperwallet Generator</h3>

				<p />
				<div class="center">
					<label class="checkerContainer">
						<input type="checkbox" bind:value={showQRCodesAlso} />
						<!-- <span class="checkmark"></span> -->
						{texts.includingQRCodes}
					</label>
					<br />
					<input type="number" bind:value={amountOfWallets} />
					<button on:click={generateAll}>{texts.generate}</button>
				</div>
			</div>
		{/if}

		<div class="center">
			{#if generationCompleted}
				{#each walletInfos as wi, index}
					<div
						class={(showQRCodesAlso && index % 2 === 0) || (!showQRCodesAlso && index % 9 === 0) && index > 1
							? "pageBreak"
							: "relax"}
					>
						<a href="https://Geo-Caching.org" target="_blank">
							<h4>Geo-Caching.org</h4>
						</a>
						<div class="small">
							<!-- {texts.congrats} -->
							<p></p>
							<b> Freedom Wallet Address (Share): </b>
							{wi.address} <br />
							{#if showQRCodesAlso}
								<canvas id={wi.canvasIDAddress} />
							{/if}
							<p></p>
							<b> Private Key (do not share): </b>
							{wi.privateKey} <br />
							<p></p>
							<b> Mnemonic Phrase (do not share): </b>
							{wi.mnemonic} <br />

							{#if showQRCodesAlso}
								<canvas id={wi.canvasIDPrivateKey} />
							{/if}
						</div>
					</div>
				{/each}
			{/if}
			<div class="noprint">
				{#if walletInfos.length === amountOfWallets}
					<div class="noprint">
						<p></p>
						<button class="b1" onclick="window.print()"
							>{texts.print}</button
						>
						<p />
						<button class="b1" onclick="window.location.reload()"
							>{texts.reload}</button
						>
					</div>

					<p><br /></p>
					Generated public wallet addresses:
					<p />
					{#each walletInfos as wiForList}
						{wiForList.address} <br />
					{/each}
					<p><br /></p>
					Generated Secrets:
					<p />
					{#each walletInfos as wiForList}
						{wiForList.mnemonic} <br />
						{wiForList.privateKey} <br />
					{/each}
				{/if}
				<p><br /></p>
			</div>
		</div>
	{/if}
</main>

<style>
	main {
		text-align: center;
		margin-left: auto;
		margin-right: auto;
		padding: 0em;
	}

	h3 {
		color: rgb(1, 111, 1);
		text-transform: uppercase;
		font-size: 2em;
		font-weight: 100;
	}

	.small {
		margin-left: 0;
		padding-left: 0;
		font-size: 9px;
	}

	@media (min-width: 640px) {
		main {
			max-width: none;
		}
	}

	@media print {
		.noprint {
			display: none;
		}

		.pageBreak {
			page-break-before: always;
		}
	}
</style>
