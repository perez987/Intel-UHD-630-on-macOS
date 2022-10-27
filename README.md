# Intel UHD Graphics 630 Coffee Lake R on macOS

<table>
       <tr><td align=center><img src=img/i9.png></td></tr>
       <tr><td><b>How to set the integrated graphics card Intel UHD Graphics 630 Coffee Lake R (9th gen. i7 and i9) in headless mode (no cable to monitor) to be used by macOS (Catalina, Big Sur, Monterey or Ventura) in computing and video encoding tasks or setting it as the main card</b></td></tr>
</table>

**Note**: based on

- `[GUIDE] General Framebuffer Patching Guide (HDMI Black Screen Problem)` by _CaseySJ_ in tonymacx86 forum
- Framebuffer patch feature of _headkaze_'s Hackintool app
- Desktop Coffee Lake chapter of the OpenCore Dortania's guide.

## Preface

Macs with an integrated graphics card (iGPU) and a dedicated one (dGPU) use the iGPU for video encoding and decoding tasks. When building a Hackintosh with both types of GPU we can find that, if the iGPU is not properly installed, video encoding can fail. When this happens, we must configure the iGPU as _headless mode_ (it is so called when it is enabled but no cable to monitor) so that the dGPU acts as main card but the iGPU is available for encoding / decoding video.

iGPU setting depends on 2 factors:

<table>
       <tr><td>Motherboard because manufacturer can put 1, 2 or 3 HDMI connectors for the iGPU on the back pannel</td></tr>
       <tr><td>Intel processor generation, different Intel generations have different iGPU</td></tr>
</table>

My PC has a Z390 Aorus Elite board with an Intel 9th gen. CPU (Coffee Lake Refresh, it is configured as Coffee Lake) with Intel UHD Graphics 630:

<table>
       <tr><td>PCI path is PciRoot(0x0)/Pci(0x2,0x0)</td></tr>
       <tr><td>Plattorm ID is 3E9B0007</td></tr>
</table>

On this board there is only one HDMI v1.4 connector for the iGPU, it corresponds to index 3 in the theoretical list of 3 internal connectors this iGPU can have:

<table>
       <tr><td>Index 1, BusID 0x00, Type HDMI (type does not matter on this port)</td></tr>
       <tr><td>Index 2, BusID 0x00, Type HDMI (type does not matter on this port)</td></tr>
       <tr><td>Index 3, BusID 0x04, Type HDMI (this is the active port, the only physical one, type is HDMI)</td></tr>
</table>

This is important when using the iGPU as a main or single card but not when using it in headless mode.

## Headless mode

- iGPU and dGPU must be enabled in BIOS with dGPU as primary
- There should be no cable between iGPU HDMI port and any type of display
- Lilu and WhatEverGreen properly installed
- SMBIOS iMac19.1.

You have to add in `DeviceProperties >> Add`:

``` xml
			<key>PciRoot(0x0)/Pci(0x2,0x0)</key>
			<dict>
				<key>AAPL,ig-platform-id</key>
				<data>AwCRPg==</data>
				<key>device-id</key>
				<data>mz4AAA==</data>
				<key>enable-metal</key>
				<data>AQAAAA==</data>
				<key>igfxfw</key>
				<data>AgAAAA==</data>
				<key>force-online</key>
				<data>AQAAAA==</data>
				<key>rps-control</key>
				<data>AQAAAA==</data>
			</dict>
```

This code has data values in Base64, in plist editors they can be seen as hexadecimal, e.g. `AwCRPg==` in Base64 (_AAPL,ig-platform-id_) = `0300913E` in hexadecimal.

With these changes you can boot from a dGPU with the iGPU well installed. To check if the VDA Decoder function is activated you can get Hackintool app (_Fully Supported_ or _Failed_ in the first System tab).

Notes:

- `device-id=9B3E000` to be displayed as `Intel UHD Graphics 630` instead of `Kabylake Unknown`
- `enable-metal=01` to enable Metal 3 in Ventura
- `force-online=01` to force online status on all displays (mandatory)
- `igfxfw=02`to force loading of Apple GuC firmware (improves IGPU performance)
- `rps-control=01` to enable RPS control patch (improves IGPU performance).

<details>
<summary>Image: iGPU as secondary card</summary>
<br>
<img src="img/iGPU as secondary card.png">
</details>

## iGPU as main card

This card can also be configured to be the main or single one, so that it outputs a signal to the monitor and also encodes video. Here's what to do.

- Enable it on the motherboard as main card: Initial Display Output `IGFX` instead of `PCIe 1 Slot` (this would be the final step)
- Lilu and WhatEverGreen properly installed
- SMBIOS iMac19.1
- Add in `config.plist >> DeviceProperties >> Add` the code below (note: `BwCbPg==` is `07009B3E` in hexadecimal):

``` xml
			<key>PciRoot(0x0)/Pci(0x2,0x0)</key>
			<dict>
				<key>AAPL,ig-platform-id</key>
				<data>BwCbPg==</data>
				<key>device-id</key>
				<data>mz4AAA==</data>
				<key>device_type</key>
				<string>VGA compatible controller</string>
				<key>enable-hdmi20</key>
				<data>AQAAAA==</data>
				<key>enable-metal</key>
				<data>AQAAAA==</data>
				<key>framebuffer-con0-busid</key>
				<data>AAAAAA==</data>
				<key>framebuffer-con0-enable</key>
				<data>AQAAAA==</data>
				<key>framebuffer-con0-pipe</key>
				<data>EgAAAA==</data>
				<key>framebuffer-con1-busid</key>
				<data>AAAAAA==</data>
				<key>framebuffer-con1-enable</key>
				<data>AQAAAA==</data>
				<key>framebuffer-con1-pipe</key>
				<data>EgAAAA==</data>
				<key>framebuffer-con2-busid</key>
				<data>BAAAAA==</data>
				<key>framebuffer-con2-enable</key>
				<data>AQAAAA==</data>
				<key>framebuffer-con2-pipe</key>
				<data>EgAAAA==</data>
				<key>framebuffer-con2-type</key>
				<data>AAgAAA==</data>
				<key>framebuffer-patch-enable</key>
				<data>AQAAAA==</data>
				<key>framebuffer-stolenmem</key>
				<data>AAAwAQ==</data>
				<key>hda-gfx</key>
				<string>onboard-1</string>
				<key>igfxfw</key>
				<data>AgAAAA==</data>
				<key>force-online</key>
				<data>AQAAAA==</data>
				<key>rps-control</key>
				<data>AQAAAA==</data>
			</dict>
```

In this way, Intel UHD 630 is well installed and works fine on macOS.

<details>
<summary>Image: iGPU as main card</summary>
<br>
<img src="img/iGPU as main card.png">
</details>

If you have KP or black screen when macOS wakes from sleep, you have to replace hda-gfx property with No-hda-gfx, this usually fixes those KPs but audio is lost through HDMI. Replace:

``` xml
	    	<key>hda-gfx</key>
	    	<string>onboard-1</string>
```

with:

``` xml
	    	<key>No-hda-gfx</key>
	    	<data>AAAAAAAAAAA=</data>
```
