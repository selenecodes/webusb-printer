<script lang="ts">
	import { Printer } from '../lib/printer';
	import '../receiptline-full';
	let printer: Printer | undefined;
	let text: string = printTemplate({
		logo: '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',
		address: 'Zuiderzeestraatweg west 55',
		city: 'Amsterdam',
		phone: 'SOMEFAKEPHONENUMBER',
		datetime: '30-10-2023 12:00',
		products: [
			{
				amount: 1,
				price: 19.95,
				description: 'Keune Style Strong Mousse 200ml'
			},
			{
				amount: 1,
				price: 17.95,
				description: 'Diversen Tangle Teezer normaal'
			}
		],
		employee: 'John Doe',
		companyName: `ACME Inc.`,
		price: {
			cashReturn: -5,
			pin: 42.90,
			discount: 0,
			subtotal: 37.90,
			total: 37.90,
			vat: [
				{
					percentage: 21,
					amount: 6.58
				},
				{
					percentage: 9,
					amount: 0
				},
				{
					percentage: 6,
					amount: 0
				}
			]
		}
	});

	async function setPrinter() {
		printer = await Printer.setup();
	}

	$: svg = receiptline.transform(text, {
		encoding: 'multilingual',
		command: 'svg'
	});

	interface PrintTemplate {
		logo: string;
		companyName: string;
		address: string;
		city: string;
		phone: string;
		datetime: string;
		products: {
			amount: number;
			price: number;
			description: string;
		}[];
		employee: string;
		price: {
			vat: {
				percentage: number;
				amount: number;
			}[];
			subtotal: number;
			discount: number;
			total: number;
			cashReturn: number;
			pin: number;
		}
	}

	function printTemplate(receipt: PrintTemplate) {
		return `{image:${receipt.logo}}



^${receipt.companyName}

${receipt.address}
${receipt.city}
${receipt.phone}
${receipt.datetime}

{w:4,8,*;t:wrap}
|"Aant |"Prijs |"Beschrijving |
-
${receipt.products.map(product => `|${product.amount} |${product.price.toFixed(2)} |${product.description} |`).join('\n')}

{w:16,6;a:right}
|Subtotaal | ${receipt.price.subtotal.toFixed(2)}|
|Korting | ${receipt.price.discount.toFixed(2)}|
|Totaal incl. BTW | ${receipt.price.total.toFixed(2)}|
|||
|cashReturn | ${receipt.price.cashReturn.toFixed(2)}|
|Pin | ${receipt.price.pin.toFixed(2)}|
|||
${receipt.price.vat.map(vat => `|BTW ${vat.percentage}% | ${vat.amount.toFixed(2)}`).join('\n')}

{w:*}
U bent geholpen door:
${receipt.employee}

Bedankt voor uw bezoek en tot ziens!`;
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
				on:click={() => printer?.print(text)}
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
