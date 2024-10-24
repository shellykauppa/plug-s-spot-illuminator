# Shelly Plug S Spot Price Illuminator

This is a [Shelly script](https://shelly-api-docs.shelly.cloud/gen2/Scripts/ShellyScriptLanguageFeatures) for [Shelly Plus Plug S](https://kb.shelly.cloud/knowledge-base/shelly-plus-plug-s) and [Shelly Plug S Mtr Gen3](https://kb.shelly.cloud/knowledge-base/shelly-plug-s-mtr-gen3) for lighting the LED according to electricity spot prices provided by the https://spot-hinta.fi API.

Key features:

- Unlimited user configurable price thresholds including color & brightness settings
- Colors configurable using CSS colors including [named colors](https://drafts.csswg.org/css-color/#named-colors)
- Switch off state distinguishable by LED brightness; each price step has a configurable brightness setting for `on` and `off`
- Price update fetched every 15 minutes (future proof for 15 minute market time)
- Offline state distinguishable by white LED color
- Listens for network / cloud service connect / disconnect events and reacts swiftly to changes in connectivity

[![Video of the script running in development mode](http://img.youtube.com/vi/lC75SAEdaqg/0.jpg)](http://www.youtube.com/watch?v=lC75SAEdaqg "Shelly Plus Plug S:n värin ohjaus sähkön hinnan perusteella")

## Usage

Install, enable and start the [script](shelly/index.js) as illustrated for example in the [Shelly script tutorial](https://shelly-api-docs.shelly.cloud/gen2/Scripts/Tutorial).

## Requirements

The Plug S LED setting has to be `switch` (as opposed to `power` or `off`) in order for the colors to work. At script initialization, the script will change mode to `switch` if not already. If the mode changes during runtime, the script will continue to run but will not perform any actions.

Plus Plug S needs a firmware newer than 1.0 (tested up to 1.4.4).

Plug S Mtr Gen3 support tested up to 1.2.3-matter22 (only fw available at this time).

## Operation

Spot price for the user configurable Spot area (default `FI`) is retrieved immediately at script startup and periodically at every even 15 minutes (hh:00, hh:15, hh:30, hh:45) after that. When price is received, the script sets the LED color according to the user configurable table of price thresholds <-> LED colors / brightness values. In order to separate between switch `on` and `off` states, a different brightness level is used. When connection to cloud service (if used at all) or local network is severed, the device LED is colored white and when connection returns the normal operations are resumed.

The default price/color map is:

```javascript
[
  { fromPriceWithTax: -1.0, color: "purple",    onBrightness: 80, offBrightness: 10 },
  { fromPriceWithTax: 0.00, color: "green",     onBrightness: 80, offBrightness: 10 },
  { fromPriceWithTax: 0.09, color: "turquoise", onBrightness: 50, offBrightness:  7 },
  { fromPriceWithTax: 0.15, color: "#FF4500",   onBrightness: 50, offBrightness: 10 }, // "orangered"
  { fromPriceWithTax: 0.25, color: "red",       onBrightness: 50, offBrightness: 10 }
]
```

## Notes

The used colors can be configured either in CSS format (i.e. `#336e12`) or using the [CSS named colors names](https://drafts.csswg.org/css-color/#named-colors). Just so you know, some of the colors look nothing like the name implies or a standard monitor shows; some trial and error might be needed to find the colors you want. The same goes for brightness levels, whitish colors may appear very bright event at low brightness levels.

Due to Shelly flavor of Javascript having no support for proper [error handling](https://shelly-api-docs.shelly.cloud/gen2/Scripts/ShellyScriptLanguageFeatures/#error-handling), a network error or API failure might result in the script execution stopping indefinitely. A support script such as https://github.com/Porssari/Shelly-client/blob/main/release/porssari-monitor-script.js should be used in addition to ensure the script is restarted if that happens.

It seems that there is a bug/feature in the device firmware 0.14.1 wherein if you visit the Home -> (Output) Settings -> LED Indication page of theinternal web server, the LED will reset to whatever state can be found there (for us it seems to be green with 80%/80% brightness). It will take until the next tick (max 15 minutes) for the color scheme to return. The same happens if the user manually changes LED colors.

With Shelly gen2 firmware v1.0.0, a code limit of 15kB will be introduced. As the plus plug illuminator code exceeds that (~17kB), code comments are kept to a minimum and other ugliness is introduced (compressed tables, short variable names). 15kB limitation was removed in 1.0.7.

Plus Plug S firmware 1.0.8 up to 1.4.4 has a bug wherein the led color cannot be changed using a Shelly.call("PLUGS_UI.SetConfig",...) so instead the rpc call is used (v2.0.0 ->). Gen3 fw 1.2.3 fixes the call but rpc call is used nevertheless to avoid branching the code.

Gen3 fw 1.2.3 limits the script's runtime memory to 7000 bytes. The css lookup takes around three times as much memory, so the css lookup is kept in program memory and used only in small slices when mapping color names to hex values (v2.1.0 ->).

For local development, set `API_HOST` variable to a local server and set `DEBUG` to true. This will make the code to poll for price changes every 5 seconds. Using `DEBUG=true` with spot-hinta.fi will soon result in your IP starting to get http 429 instead of data.

## Mock server

This repository includes a mock node.js server that simulates the [api.spot-hinta.fi/JustNow](https://api.spot-hinta.fi/swagger/ui#/(JSON)%20Current%20hour%20(or%20one%20of%20the%20next%20hours)/JustNow) endpoint (for the parts we're interested) and returns random parametrizable prices. To run the server in http://localhost:4390 you can just run the following in the repository root:

```shell-script
npm install
npm run serve

```

## TODO

Known features and improvements that could be done. Pull requests are welcome!

- [ ] Alternative price map color format using the Shelly internal values (0...100, 0...100, 0...100) to make it easier to transform the values from the cloud / app to script.
