
<script lang="ts">
	import { browser } from '$app/environment';
	import { Buffer } from 'buffer'
	import '../receiptline-full';

    // Vendor and Product ID to select
	const STAR_VENDOR_ID = 0x0519; // STAR MICRONICS CO., LTD
	const STAR_TSP143_PRODUCT_ID = 0x0003; // TSP143III
	let endpoints: Record<USBEndpoint['direction'], USBEndpoint> | undefined;
	let printer: USBDevice | undefined;


	/**
	 * Configures the given printer.
	 * @description Configures the given printer by opening a connection,
	 * selecting the first configuration and claiming the first interface
	 * and lastly it returns the endpoints of the first interface
	 * @param printer - The printer to configure
	 * @returns The endpoints of the first interface
	 */
	async function configurePrinter(printer: USBDevice): Promise<Record<USBEndpoint['direction'], USBEndpoint>> {
		// Open the printer and select the first configuration
		await printer.open();
		await printer.selectConfiguration(1);

		const interfaces = printer.configuration?.interfaces;
		if (!interfaces) throw new Error('No interfaces found');
		// NOTE: We might want to check if the printer has multiple interfaces and select the correct one
		// currently this implementation works fine though*
        const firstInterface = interfaces[0];
        await printer.claimInterface(firstInterface.interfaceNumber);
        const endpoints = firstInterface.alternate.endpoints.reduce((endpoints, endpoint) => ({
            ...endpoints,
            [endpoint.direction]: endpoint,
        }), {}) as Record<USBEndpoint['direction'], USBEndpoint>;

        if (!endpoints.in || !endpoints.out) throw new Error('No endpoints found');
        return endpoints;
	}

	/**
	 * Request access to a USB printer.
	 * @description Requests a printer from the browser and saves it to the printer variable
	 * @note For more info about the WebUSB api see https://developer.mozilla.org/en-US/docs/Web/API/WebUSB_API
	 */
	async function getPrinter() {
		const device = await navigator.usb.requestDevice({
			filters: [
				{
					vendorId: STAR_VENDOR_ID,
					productId: STAR_TSP143_PRODUCT_ID
				}
			]
		});

		if (!device) throw new Error('No printer found');
		printer = device;
        endpoints = await configurePrinter(printer);
	}

	/**
	 * Creates the prefix that's used before a base64 data url.
	 * @description The prefix is used to tell the browser what type of data the url contains
	 * we use this to tell the browser that the url contains a png or svg.
	 * @note For more info about data urls see https://developer.mozilla.org/en-US/docs/Web/HTTP/Basics_of_HTTP/Data_URIs
	 * @note We also use this to subtract this prefix from the base64 string when printing the png
	 * @param type - The type of the meta tag to create
	 * @returns The meta tag
	 */
	function createDataURLPrefix(type: 'png' | 'svg+xml') {
		return `data:image/${type};base64,`
	}

	/**
	 * Gets the width and height of the given svg
	 * @description Converts an svg string to an xml document and gets the width and height from the xml document
	 * @param svg - The svg (as a string) to get the width and height of
	 * @returns The width and height of the svg
	 */
	function getSVGSize(svg: string): { width: number, height: number } {
		const parser = new DOMParser();
		const xmlDoc = parser.parseFromString(svg, "text/xml");
		const height = xmlDoc.documentElement.getAttribute('height');
		const width = xmlDoc.documentElement.getAttribute('width');
		if (!height || !width) throw new Error('No height or width found');
		return {
			height: parseInt(height),
			width: parseInt(width)
		}
	}

	/**
	 * Creates a canvas element with the given width and height
	 * @param width - The width of the canvas
	 * @param height - The height of the canvas
	 * @returns The canvas element
	 */
	function createCanvas(width: number, height: number): HTMLCanvasElement {
		const canvas = document.createElement( "canvas" );
		canvas.setAttribute("width", width.toString());
		canvas.setAttribute("height", height.toString());
		return canvas;
	}

	/**
	 * Creates an image element with the given svg, width and height
	 * @param svg - The svg (as a string) to create the image from
	 * @param width - The width of the image
	 * @param height - The height of the image
	 * @returns The image element
	 */
	function createImage(svg: string, width: number, height: number): HTMLImageElement {
		const img = document.createElement("img");
		const svgMetaTag = createDataURLPrefix('svg+xml')
		img.setAttribute("src", svgMetaTag + btoa(svg));
		img.setAttribute("width", width.toString());
		img.setAttribute("height", height.toString());
		return img
	}

	/**
	 * Creates a png from the given receiptline template and calls the callback with the png as a base64 string
	 * @param receiptlineTemplate - The receiptline template to create the png from
	 * @param callback - The callback to call with the png as a base64 string
	 * @note For info about receiptline templates see https://github.com/receiptline/receiptline/tree/master
	 */
	function createPng(receiptlineTemplate: string, callback: (png: string) => void): void {
		const svg = receiptline.transform(receiptlineTemplate, {
			encoding: 'multilingual',
			command: 'svg'
		});

		const { width, height } = getSVGSize(svg);
		const canvas = createCanvas(width, height);
		const ctx = canvas.getContext( "2d" );
		if (!ctx) throw new Error('No canvas context found');

		const img = createImage(svg, width, height);
		img.onload = function() {
			ctx.drawImage(img, 0, 0);
			const base64Png = canvas.toDataURL("image/png")
			const pngMetaTag = createDataURLPrefix('png');
			const base64PngWithoutMetaTag = base64Png.replace(pngMetaTag, '');
			callback(base64PngWithoutMetaTag)
		}
	}

	/**
	 * Prints the given png to the printer
	 * @param png - The png to print as a base64 string
	 * @note Star TSP1xx printers only support printing raster images without a print driver installed.
	 * this means that the printer can only print pngs and not svgs requiring us to convert the svg to a png
	 * in the browser using a canvas.
	 * @note Webbrowser support: We do not support IE due to it not supporting this conversion from svg -> png
	 */
	async function print(png: string) {
		if (!printer) throw new Error('No printer found');
		if (!endpoints) throw new Error('No endpoints found');
		const command = receiptline.transform(`{image:${png}}`, {
			cutting: true,
			command: 'stargraphic'
		});

		const binaryData = Buffer.from(command, 'binary');
		printer.transferOut(endpoints.out.endpointNumber, binaryData)
	}

	const EXAMPLE_RECEIPT = `{image:iVBORw0KGgoAAAANSUhEUgAAAQAAAAA8AgMAAAD004yXAAAACVBMVEVwAJsAAAD///+esS7BAAAAAXRSTlMAQObYZgAAAZtJREFUSMftlkGOwyAMRW0J76kE97GlZu9KcP+rzCekKak6bWdGmrZS6KIG/7yAbQhEe+tN6qUpDZ1KPHT8SnhpeaP6FlA2wicBsgOeBpTF6gDfZHgj9Jt1sAMeAYYE/RFQngYMuX49gD8fMObjkwB+++h+DeB3x/qdvfD/AP4QwPXX+ceAUdV6Y7K/3Vr7rWhv73ZN9XVHzOUdlm6v9ay3pM1eLT+Ppv63BWjz8jJU0vpUWgGs5yfihrN451G+lq7i2bkF8Bagw3Qv0hEQKGTPPjnGxJtZrSIcJy5ciJ3JStbarjrgwOtVK7Z1iLFOOgOSR3WstTqMqCcVFlITDnhhNhMKzJNIA3iEBr1uaAfArA7bSWdAkRrIsJSMrwXbJBowEhogwe9F+NilC8D1oIyg43fA+1WwKpoktXUmsiOCjxFMR+gEBfwcaJ7qDBBWNTVSxKsZZhwkUgkppEiJ1VOIQdCFtAmNBQg9QWj9POQWFW4Bh0HVipRIHlpEUVzUAmslpgQpVcS0SklcrBqf03u/UvVhLd8H5Hffil/ia4Io3warBgAAAABJRU5ErkJggg==}

Ichigaya Terminal
1-Y-X Kudan, Chiyoda-ku
02-07-2021 21:00
{border:line; width:30}
^RECEIPT
{border:space; width:*,2,10}
BEER                   | 2|     13.00
CHIDORI                | 2|    172.80
-------------------------------------
{width:*,20}
^TOTAL             |          ^185.80
CASH               |           200.00
CHANGE             |            14.20
{code:20210207210001; option:48,hri}`
</script>

<button on:click={getPrinter}>Request usb device</button>
<button on:click={() => createPng(EXAMPLE_RECEIPT, (png) => print(png))}>
	Print
</button>
