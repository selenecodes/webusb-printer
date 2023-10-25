
<script lang="ts">
	import { Printer } from "./printer";
	let printer: Printer | undefined;

	async function setPrinter() {
		printer = await Printer.setup();
	}

	// function printTemplate(logo: string) {
	// 	printer?.print(`{image:${logo}}
		
	// 	`);
	// }

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

<button on:click={setPrinter}>Request usb device</button>
{#if printer?.printer}
	<button on:click={() => printer?.print(EXAMPLE_RECEIPT)}>
		Print
	</button>
{/if}