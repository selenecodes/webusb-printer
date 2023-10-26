
<script lang="ts">
	import { EXAMPLE_RECEIPT_1 } from "../lib/example-receipts";
	import { Printer } from "../lib/printer";
	let printer: Printer | undefined;
	let text: string = EXAMPLE_RECEIPT_1;

	async function setPrinter() {
		printer = await Printer.setup();
	}

	// function printTemplate(logo: string) {
	// 	printer?.print(`{image:${logo}}
		
	// 	`);
	// }
</script>

<div style="display:grid; grid-auto-flow: row; gap: 1em;">
	<button on:click={setPrinter}>Request usb device</button>
	{#if printer}
		<textarea rows="20" cols="80" bind:value={text}></textarea>
		<button on:click={() => printer?.print(text)}>
			Print
		</button>
	{/if}
</div>
