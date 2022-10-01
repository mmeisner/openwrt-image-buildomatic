# OpenWRT Image Buildomatic

This shell script uses the official
[OpenWrt Image Builder](https://openwrt.org/docs/guide-user/additional-software/imagebuilder)
to quickly generate/build an OpenWRT device image with extra pre-installed
packages, files or configuration included in the final image.

This means that to build your very own, let's say Raspberry Pi 4,
OpenWRT image, you just need these three lines:

```
    git clone https://github.com/mmeisner/openwrt-image-buildomatic
    cd openwrt-image-buildomatic
    ./oi-build -b -c rpi-4
```
...and then after a minute you will have an image you can write to an SD-card,
insert it into your RPi4 and run your Internet connection with 940Mbps
throughput, with Wireguard.

See the device configuration files in the
[configs](https://github.com/mmeisner/openwrt-image-buildomatic/tree/main/configs)
folder for how simple the configuration is.

## The OpenWRT Image Builder

The *OpenWRT Image Builder* is like the normal OpenWRT build system except
it does not build from source code but downloads a pre-built binary image
and packages and combines those into a new image that can be used to update
your OpenWRT device; Instead of taking hours to build an image from source,
it takes mere seconds to build an image - depending on your download speed.

The OpenWRT image builder is already very easy to use, so the only thing that
this script adds, is automated download and execution of the image builder
&mdash; plus a convenient way to specify your configuration in a simple
textual file (actually merely a shell script with a few variables).
In the configuration file you can specify:

  - **`RELEASE`** is the release to use, e.g. `22.03`, `21.02-rc1`, `19.07`, `snapshot`, or whatever
  - **`TARGET`** is the CPU architecture such as *`x86`*,
    *`ath79`* (for Archer C7), *`bcm27xx`* (for Raspberry PI), etc.
  - **`PROFILE`** is the device name to make an image for e.g. *`rpi4`* or *`archer-c7-v2`*
  - **`PACKAGES`** is a list of additional packages to include in the image.
    Full list per release is here: [OpenWrt Packages](https://openwrt.org/packages/start)
  - **`FILES`** is the path to a directory containing additional files you
    want to add to the final image. The directory should be organized as
    a normal rootfs. It could contain some useful home-made scripts or
    `uci-defaults` files for predefined configuration at first boot (see
    [UCI defaults](https://openwrt.org/docs/guide-developer/uci-defaults))

Please note that as stated on the
[OpenWrt Image Builder](https://openwrt.org/docs/guide-user/additional-software/imagebuilder)
page, *prebuilt snapshot images do not come with any web interface or GUI*.
This means that `PACKAGES` should at least contain `luci` if you want the web GUI!

Here are some of the OpenWRT [Release Builds](https://openwrt.org/releases/start)
and devices that the script works with (as of Sep 2022):

  - Releases:
    - *snapshot*    (snapshot, next upcoming release)
    - *22.03*
    - *21.02*
    - *19.07*  
  - Some popular devices:
    - *Raspberry Pi 4*
    - *Archer C7*

OpenWRT [Index of /releases/](https://downloads.openwrt.org/releases/)

# Other Useful OpenWRT Projects and Resources

This section lists various other resources on the Internet which can be useful
with regard to installing or upgrading your OpenWRT router.

* [uciparse · PyPI](https://pypi.org/project/uciparse/)
  These tools were written to ease OpenWRT upgrades, making it easier to see
  the differences between two config files. As of this writing (mid-2020),
  OpenWRT upgrades often don't normalize upgraded config files in the same
  way from version to version

## Post Install Actions

* [[OpenWrt Wiki] UCI defaults](https://openwrt.org/docs/guide-developer/uci-defaults)
  describes how to setup some defaults at first boot
* [[OpenWrt Wiki] Opkg extras](https://openwrt.org/docs/guide-user/advanced/opkg_extras)
  provides a way to reinstall user-installed packages after an upgrade
* [[OpenWrt Wiki] UCI extras](https://openwrt.org/docs/guide-user/advanced/uci_extras)
  describes how to add files to `/etc/uci-defaults` which can configure
  your OpenWRT install after an initial install or upgrade.
* [[OpenWrt Wiki] Opkg extras](https://openwrt.org/docs/guide-user/advanced/opkg_extras)
  extends `opkg` to &mdash; among other things &mdash; handle user-installed packages
* [How to automate upgrade? - For Developers - OpenWrt Forum](https://forum.openwrt.org/t/how-to-automate-upgrade/72636/12)
  has some pointers on how to re-install user installed packages when updating.

## OpenWRT Remote Control Projects

  * [jumpscale7/openwrt-remote-manager: GitHub](https://github.com/jumpscale7/openwrt-remote-manager)
    A Python module for remotely managing an OpenWRT instance. 22 stars, 6 forks, last active 2015.
    Requires package `luci-mod-rpc` installed on your OpenWRT
  * [fbradyirl/openwrt-luci-rpc](https://github.com/fbradyirl/openwrt-luci-rpc)
    * Good docs, 17 stars, 10 forks, last active 2021-Mar
    * Only an RPC transport. All functions must be implemented on top of that.  
    * [openwrt-luci-rpc · PyPI](https://pypi.org/project/openwrt-luci-rpc/)
  * [Noltari/python-ubus-rpc](https://github.com/Noltari/python-ubus-rpc).
    * Python module implementing an interface to the OpenWrt ubus RPC API.
    * Looks pretty sweet. 0 stars, 0 forks, 2021-feb-17.
    * Limited number of functions implemented.
    * [openwrt-ubus-rpc · PyPI](https://pypi.org/project/openwrt-ubus-rpc/)
  * [jianingy/openwrt-wac: GitHub](https://github.com/jianingy/openwrt-wac)
    A simple command line tool for managing groups of OpenWRT based routers.
    * 3 stars, 1 fork, last active 2016
    * [openwrt-wac · PyPI](https://pypi.org/project/openwrt-wac/)


## Some Useful OpenWRT Settings

Disable DHCP server on LAN: `uci set dhcp.lan.ignore=1 && uci commit && /etc/init.d/dnsmasq restart`

# Raspberry Pi 4

OpenWRT uses a single (main) partition for both the `/rom` and `rootfs_data`
and uses a special `mtdsplit` driver to make this sub-partitioning.
The `/rom` contains a squashfs filesystem with kernel and rootfs and
`rootfs_data` is where the writable `overlayfs` is mounted:
[[OpenWrt Wiki] The OpenWrt Flash Layout](https://openwrt.org/docs/techref/flash.layout)

For some reason, on a Raspberry Pi 4, this partition is by default only 104MB.
This leaves only around 70MB writable space. Run-time re-partitioning is quite
complicated (contact me if you know how to do it) so that leaves us with
the only option of building the image with a pre-determined size.
The `oi-build` script takes the value of `CONFIG_TARGET_ROOTFS_PARTSIZE` as
the size of the main partition. Make sure that you don't make it larger than
the SD-card you are using.

NOTE: Contrary to what the OpenWRT Wiki says,
`CONFIG_TARGET_ROOTFS_PARTSIZE` is ignored by the "make" command-line,
and you have to set it in the `.config` file instead.

See additional info here from the forum:
  * [Building a x86-based image and need more disk space in the image - For Developers - OpenWrt Forum](https://forum.openwrt.org/t/building-a-x86-based-image-and-need-more-disk-space-in-the-image/2585/5)
  * [X86 sysupgrade not working - For Developers - OpenWrt Forum](https://forum.openwrt.org/t/x86-sysupgrade-not-working/37869)

https://forum.openwrt.org/t/x86-sysupgrade-not-working/37869

## Raspberry Pi 4 Network Performance Tuning

  * [(15) BIFRÖST Part 2 - RPi 4B Hybrid Bridging Router - Tuning Linux Network for Gigabit Line Speed | LinkedIn](https://www.linkedin.com/pulse/bifr%C3%B6st-rpi-4b-hybrid-routing-bridge-tuning-linux-network-corner/?trk=read_related_article-card_title)
    very detailed article about tuning RPi4 network performance using IRQ
    CPU pinning and other tricks. Kind of a hardwired ues of `irqbalance`


# Similar projects

  * [Github ansemjo/openwrtbuilder](https://gist.github.com/ansemjo/cb41677a76f1c063527744438b03b932)
    *OpenWRT builder - using the imagebuilder to compile a custom image*
    is a small, simple shell script that does pretty much the same as this script
    although with command-line arguments instead of a configuration file.

----
Convert this markdown doc to HTML with pandoc: 

pandoc --toc --self-contained -t html -o README.html README.md
