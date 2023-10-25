# Star TSP100 WebUSB printing

This repository contains an example SvelteKit project that allows for printing to a Star TSP100III printer (TSP143III etc) using WebUSB. It uses the following dependencies:

- buffer (to use Node.JS' `buffer.from`)
- receiptline (to format and create the printer commands)
- 

## Troubleshooting

### My browser doesn't print

This could be correct, ensure you're using one of the browsers that support
the WebUSB standard. A list of browsers can be found on [Caniuse WebUSB](https://caniuse.com/webusb)

### Printing doesn't work on Windows

For windows you'll need to install [Zadig](https://zadig.akeo.ie/) to overwrite the printer driver from usbprint.sys to winusb.sys.

#### Steps

1. Run Zadig as administrator
2. In the menubar select Options -> List all devices
3. Select the Star printer
4. Ensure the right dropdown menu shows WinUSB.
5. Click "Reinstall WCID Driver"

### ImportError

We use browserify (`npm run browserify`) to convert the `receiptline` dependency from Node.JS to plain JS. This should run at install time because of the `postinstall` script in the package.json, however if this is not the case for you please run it manually.