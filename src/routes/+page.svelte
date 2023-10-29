<script lang="ts">
	import { EXAMPLE_RECEIPT_1 } from '../lib/example-receipts';
	import { Printer } from '../lib/printer';
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

	function printTemplate(logo: string) {
		const template = `{image:${logo}}}`
		// TODO: create our own receipt template which we fill in here.
		printer?.print(text);
	}
</script>

<div class="p-4">
	<h1 class="text-2xl">Star TSP143III WebUSB Printing example</h1>
	<p>
		This is an example of how to use WebUSB to print to a Star TSP143III receipt printer.<br/>
		The repository for this example can be found on <a class="text-blue-500" href="https://github.com/selenecodes/webusb-printer">Github</a>.
	</p>
	<details class="cursor-pointer">
		<summary class="text-xl">Troubleshooting</summary>
		<h3 class="text-lg">My browser doesn't print</h3>
		<p>
			This could be correct, ensure you're using one of the browsers that support the WebUSB standard.<br />
			A list of browsers can be found on <a class="text-blue-500" href="https://caniuse.com/webusb">Caniuse WebUSB</a>.
		</p>
		<h3 class="text-lg">Printing doesn't work on Windows</h3>
		<p>
			For windows you'll need to install <a class="text-blue-500" href="https://zadig.akeo.ie/">Zadig</a> to overwrite the printer driver from usbprint.sys to winusb.sys.
		</p>
		<h4 class="font-medium">Steps:</h4>
		<ol class="list-disc ml-2">
			<li>Run Zadig as administrator</li>
			<li>In the menubar select Options &rarr; List all devices</li>
			<li>Select the Star printer</li>
			<li>Ensure the dropdown menu shows WinUSB.</li>
			<li>Click "Reinstall WCID Driver"</li>
		</ol>
	</details>

	<div class="flex flex-col gap-2 pt-2">
		<div class="flex gap-2">
			<button on:click={setPrinter} class="border p-2 rounded">Request usb device</button>
			<button
				on:click={() => printer?.openCashDrawer()}
				disabled={!printer}
				class="border p-2 rounded disabled:text-gray-500 disabled:cursor-not-allowed"
			>
				Open cash drawer
			</button>
			<button
				on:click={() => printTemplate('TODO: Add a base64 encoded png logo here')}
				disabled={!printer}
				class="border p-2 rounded disabled:text-gray-500 disabled:cursor-not-allowed"
			>
				Print
			</button>
		</div>
		<div class="flex gap-8">
			<textarea rows="30" bind:value={text} class="border p-2 rounded basis-1 grow" />
			<div class="basis-1 grow">{@html svg}</div>
		</div>
	</div>
</div>
