# Intel UHD Graphics 630 Coffee Lake R on macOS

<table>
       <tr><td align=center><img src=i9.png></td></tr>
       <tr><td><b>How to set the integrated graphics card Intel UHD Graphics 630 Coffee Lake R (i7-9700) in headless mode (no cable to monitor) to be used by macOS (Catalina, Big Sur or Monterey) in computing and video encoding tasks or setting it as the main card</b></td></tr>
</table>

**Note**: based on

- `[GUIDE] General Framebuffer Patching Guide (HDMI Black Screen Problem)` by _CaseySJ_ in tonymacx86 forum
- Framebuffer patch feature of _headkaze_'s Hackintool app
- Desktop Coffee Lake part of the OpenCore Dortania's guide.

## Preface

Macs with an integrated graphics card (iGPU) and a dedicated one (dGPU) use the iGPU for video encoding and decoding tasks. When building a Hackintosh with both types of GPU we can find that, if the iGPU is not properly installed, video encoding can fail. When this happens, we must configure the iGPU as _headless mode_ (it is so called when it is enabled but no cable to monitor) so that the dGPU acts as main card but the iGPU is available for encoding / decoding video.

iGPU setting depends on 2 factors:

<table>
       <tr><td>motherboard because manufacturer can put 1, 2 or 3 HDMI connectors for the iGPU on the back pannel</td></tr>
       <tr><td>Intel processor generation, different Intel generations have different iGPU</td></tr>
</table>

My PC has a Z390 Aorus Elite board with an Intel 9th gen. CPU (Coffee Lake Refresh, it is configured as Coffee Lake) with Intel UHD Graphics 630:

<table>
       <tr><td>PCI path is PciRoot(0x0)/Pci(0x2,0x0)</td></tr>
       <tr><td>Plattorm ID is 3E9B0007</td></tr>
       <tr><td>Device ID it is 3E910000</td></tr>
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

```
	<key>PciRoot(0x0)/Pci(0x2,0x0)</key>
	<dict>
		<key>AAPL,ig-platform-id</key>
		<data>AwCRPg==</data>
	</dict>
```

This code has data values in Base64, in plist editors they can be seen as hexadecimal, e.g. `AwCRPg==` in Base64 (_AAPL,ig-platform-id_) = `0300913E` in hexadecimal.

With these changes you can boot from a dGPU with the iGPU well installed. To check if the VDA Decoder function is activated you can see it in Hackintool app (Fully Supported or Failed in the first System tab).

## iGPU as main card

This card can also be configured to be the main or single one, so that it outputs a signal to the monitor and also encodes video. Here's what to do.

- Enable it on the motherboard as main card: Initial Display Output `IGFX` instead of `PCIe 1 Slot` (this would be the final step)
- Lilu and WhatEverGreen properly installed
- SMBIOS iMac19.1
- Add in config.plist boot-args: `igfxonln=1`
- Add in `config.plist >> DeviceProperties >> Add` the code below (note: `BwCbPg==` is `07009B3E` in hexadecimal):

```
	<key>PciRoot(0x0)/Pci(0x2,0x0)</key>
	<dict>
		<key>AAPL,ig-platform-id</key>
		<data>BwCbPg==</data>
		<key>framebuffer-patch-enable</key>
		<data>AQAAAA==</data>
		<key>framebuffer-con0-enable</key>
		<data>AQAAAA==</data>
		<key>framebuffer-con1-enable</key>
		<data>AQAAAA==</data>
		<key>framebuffer-con2-enable</key>
		<data>AQAAAA==</data>
		<key>framebuffer-con0-alldata</key>
		<data>AQAJAAAEAADHAwAA</data>
		<key>framebuffer-con1-alldata</key>
		<data>AgAKAAAEAADHAwAA</data>
		<key>framebuffer-con2-alldata</key>
		<data>AwQIAAAIAADHAwAA</data>
		<key>framebuffer-stolenmem</key>
		<data>AAAwAQ==</data>
		<key>hda-gfx</key>
		<string>onboard-1</string>
		<key>name</key>
		<string>Intel UHD Graphics 630</string>
	</dict>
```
If you have KP or black screen when macOS wakes from sleep, you have to replace hda-gfx property with No-hda-gfx, this usually fixes those KPs but audio is lost through HDMI. Replace:

```
	    	<key>hda-gfx</key>
	    	<string>onboard-1</string>
```

with:

```
	    	<key>No-hda-gfx</key>
	    	<data>AAAAAAAAAAA=</data>
```

In this way, Intel UHD 630 is well installed and works fine on macOS.
