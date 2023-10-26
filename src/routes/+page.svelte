
<script lang="ts">
	import { EXAMPLE_RECEIPT_1 } from "../lib/example-receipts";
	import { Printer } from "../lib/printer";
	import '../receiptline-full';
	let printer: Printer | undefined;
	let text: string = EXAMPLE_RECEIPT_1;

	async function setPrinter() {
		printer = await Printer.setup();
	}

	$: svg = receiptline.transform(text, {
		encoding: 'multilingual',
		command: 'svg'
	});

	// function printTemplate(logo: string) {
	// 	printer?.print(`{image:${logo}}
		
	// 	`);
	// }
</script>

<div class="flex flex-col gap-2 p-4">
	<div class="flex gap-2">
		<button on:click={setPrinter} class="border p-2 rounded">Request usb device</button>
		<button on:click={() => printer?.print(text)} disabled={!!!printer} class="border p-2 rounded disabled:text-gray-500 disabled:cursor-not-allowed">
			Print
		</button>
	</div>
	<div class="flex gap-8">
		<textarea rows="30" bind:value={text} class="border p-2 rounded basis-1 grow"/>
		<div class="basis-1 grow">{@html svg}</div>
	</div>
</div>
