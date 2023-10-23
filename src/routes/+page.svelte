
<script lang="ts">
	import { browser } from '$app/environment';
	import { Buffer } from 'buffer'
	import '../receiptline-full';

    // Vendor and Product ID to select
	const STAR_VENDOR_ID = 0x0519; // STAR MICRONICS CO., LTD
	const STAR_TSP143_PRODUCT_ID = 0x0003; // TSP143III


	async function configurePrinter(printer: USBDevice) {
		await printer.open();
		await printer.selectConfiguration(1);

		const interfaces = printer.configuration?.interfaces;
		if (!interfaces) throw new Error('No interfaces found');
        const firstInterface = interfaces[0];
        await printer.claimInterface(firstInterface.interfaceNumber);
        const endpoints = firstInterface.alternate.endpoints.reduce((endpoints, endpoint) => ({
            ...endpoints,
            [endpoint.direction]: endpoint,
        }), {}) as Record<USBEndpoint['direction'], USBEndpoint>;

        if (!endpoints.in || !endpoints.out) throw new Error('No endpoints found');
        return endpoints;
	}

	const doc = `{image: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}`;
	// // receiptline.transform(doc, {
	// //     "command"
	// // })
	// // printer example


	if (browser) {
		// (async () => {
		//     const device = findByIds(STAR_VENDOR_ID, STAR_TSP143_PRODUCT_ID);
		//     console.log(device);
		//     await device.open();
		//     // Uses blocking calls, so is async
		//     // const webDevice = await WebUSBDevice.createInstance(device);
		//     // const interfaces = webDevice.configuration.interfaces;
		//     if (!device?.interfaces) return
		//     device.interfaces.forEach((iface) => {
		//         iface.setAltSetting(iface.altSetting, () => {
		//             try {
		//                 iface.claim();
		//                 iface.endpoints.forEach((endpoint) => {
		//                     if (endpoint.direction === 'out') {
		//                         endpoint?.transfer(buffer, (error, actual) => {
		//                             console.log('outgoing transfer');
		//                             if (error) {
		//                                 console.log(error);
		//                             }
		//                         });
		//                     }
		//                     if (endpoint.direction === 'in') {
		//                         endpoint.transfer(1024, (error, actual) => {
		//                             console.log('incoming transfer');
		//                             if (error) {
		//                                 console.log(error);
		//                             }
		//                         });
		//                     }
		//                 });
		//             } catch (error) {
		//                 console.log(error);
		//             }
		//         });
		//     });
		// })();
	}

	async function getPrinter() {
		const printer = await navigator.usb.requestDevice({
			filters: [
				{
					vendorId: STAR_VENDOR_ID,
					productId: STAR_TSP143_PRODUCT_ID
				}
			]
		});

		if (!printer) throw new Error('No printer found');
        const endpoints = await configurePrinter(printer);
		const command = receiptline.transform(doc, {
			// encoding: 'multilingual',
			cutting: true,
			command: 'stargraphic'
		});

		const binaryData = Buffer.from(command, 'binary');
		
		printer.transferOut(endpoints.out.endpointNumber, binaryData)
	}
</script>

<button on:click={getPrinter}>Request usb device</button>
