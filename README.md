# VPN Hotspot

[![CircleCI](https://circleci.com/gh/Mygod/VPNHotspot.svg?style=svg)](https://circleci.com/gh/Mygod/VPNHotspot)
[![API](https://img.shields.io/badge/API-21%2B-brightgreen.svg?style=flat)](https://android-arsenal.com/api?level=21)
[![Releases](https://img.shields.io/github/downloads/Mygod/VPNHotspot/total.svg)](https://github.com/Mygod/VPNHotspot/releases)
[![Codacy Badge](https://api.codacy.com/project/badge/Grade/e70e52b1a58045819b505c09edcae816)](https://www.codacy.com/app/Mygod/VPNHotspot?utm_source=github.com&amp;utm_medium=referral&amp;utm_content=Mygod/VPNHotspot&amp;utm_campaign=Badge_Grade)
[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)

Connecting things to your VPN made simple. Share your VPN connection over hotspot or repeater. (**root required**)  
<a href="https://play.google.com/store/apps/details?id=be.mygod.vpnhotspot" target="_blank"><img src="https://play.google.com/intl/en_us/badges/images/generic/en-play-badge.png" height="60"></a>
or <a href="https://labs.xda-developers.com/store/app/be.mygod.vpnhotspot" target="_blank">XDA Labs</a>

This app is useful for:

* Connecting things that don't support VPN like Chromecasts behind corporate firewalls;
* Setting up [gapps](https://support.google.com/pixelphone/answer/7158475) behind corporate firewalls;
* Connecting to your mobile hotspot but you're not bothered to set up VPN on your device;
* Identifying, monitoring and blocking (unwanted) clients;
* Bypassing tethering limits by:
    - (recommended) Use this app with a real VPN/socksifier; (ad-blocker may not work)
    - (might or might not work based on your carrier) Use this app with TTL editor like
      [TTL Master](https://github.com/TTLMaster/TTLMaster);
    - Try your luck and simply use this app.

P.S. You can also do the similar on [Windows](https://www.expressvpn.com/support/vpn-setup/share-vpn-connection-windows/),
[Mac](https://www.expressvpn.com/support/vpn-setup/share-vpn-connection-mac/),
and [iOS](http://www.tetherme.net/).
I don't know about you but I can't get my stupid Windows 10 to work with
[hosted network](https://msdn.microsoft.com/en-us/library/windows/desktop/dd815243(v=vs.85).aspx)
now that they introduced this
[Mobile hotspot](https://support.microsoft.com/en-us/help/4027762/windows-use-your-pc-as-a-mobile-hotspot).

## Settings and How to Use Them

Default settings are picked to suit general use cases and maximize compatibility but it might not be optimal for battery
 life.

### Upstream

* Upstream network interface: Main upstream used to reroute traffic. Leave blank for auto detect system VPN.
  Put `none` (or other similarly invalid entries) to suppress tethering VPN.
  Put your interface name for tethering WireGuard.
* Fallback upstream: Fallback upstream is used when some VPN leave certain routes fallback to default network interface.
  Leave blank for auto detect. Put `none` (or other similarly invalid entries) to forbid falling back.
  Put other interface name if you feel like it.
* IP Masquerade: Source address/port from downstream packets will be remapped. Leave on if you don't know what it does.
  I find turning this option off sometimes works better for dummy VPNs like ad-blockers and socksifiers (like
  Shadowsocks). But you should never turn it off for real VPNs like OpenVPN, WireGuard, etc.

### Downstream

* Disable IPv6 tethering: Turning this option on will disable IPv6 for system tethering. Useful for stopping IPv6 leaks
  as this app currently doesn't handle IPv6 VPN tethering (see [#6](https://github.com/Mygod/VPNHotspot/issues/6)).
* Fallback DNS server[:port]: Only used when a DNS server isn't found on the upstream interface.
* Enable DHCP workaround: Only used if your device isn't able to get your clients IP addresses with VPN on.

### Misc

* Keep Wi-Fi alive: Acquire Wi-Fi locks when repeater, temporary hotspot or system VPN hotspot is activated.
   - Choose "System default" to save battery life;
   - Choose "On" (default) if repeater/hotspot turns itself off automatically or stops working after a while;
   - Choose "High Performance Mode" to minimize packet loss and latency (will consume more power).
* Start repeater on boot: Self explanatory.
* Network status monitor mode: This option controls how the app monitors connected devices as well as interface changes
  (when custom upstream is used).
  Requires restarting the app to take effects. (best way is to go to app info and force stop)
   - Netlink monitor: Use Linux netlink mechanism, most battery efficient but may not work with SELinux enforcing mode.
     Sometimes auto fallbacks to Netlink monitor with root and Poll.
   - Netlink monitor with root: Same as above but runs netlink as root. This option works well with SELinux enforcing mode
     but might still be bugged on devices heavily modified by OEM and/or carriers. Sometimes auto fallbacks to Poll.
   - Poll: (default) Update network information manually every second. Least battery efficient but it should work on most
     devices. Recommended to switch to other modes if possible.


## Q & A

Search the [issue tracker](https://github.com/Mygod/VPNHotspot/issues) for more.

### [What changes exactly can this app do to my system? (and how to revert them)](https://github.com/Mygod/VPNHotspot/issues/8#issuecomment-448529512)

### Failed to create group due to internal error/repeater shuts down after a while?

This could caused by the Wi-Fi channel you selected is no longer available, due to:

1. Your device doesn't support operating on this channel, or
2. There is some nearby Wi-Fi direct device that broadcasted that it can't operate on the channel you picked.

For maximum stability, you need to set channel = 0 so that your device will pick a channel automatically.
You can also use WPS to connect your 2.4GHz-only device to force the repeater to switch from 5GHz to 2.4GHz for this time.

### No root?

This app requires root to make changes to iptables and ip rule. This restriction is imposed by Linux kernel.

Without root, you can only:

* View connected devices for system tethering and monitor them;
* Create a hotspot that has only LAN connections;
* Toggle tether switches if you can't do it already;
* Play around with settings and the user interface in general;
* Alternatively you can use try these apps (requires manual proxy configuration or client apps) for normal repeater
  tethering/bypassing tethering limits: (note: these apps are neither free nor open source)
  * [PdaNet+](https://play.google.com/store/apps/details?id=com.pdanet)
  * [NetShare-no-root-tethering](https://play.google.com/store/apps/details?id=kha.prog.mikrotik)

## Private APIs used / Assumptions for Android customizations

_a.k.a. things that can go wrong if this app doesn't work._

This is a list of stuff that might impact this app's functionality if unavailable.
This is only meant to be an index. You can read more in the source code.

API light grey list:

* (since API 24) [`Landroid/bluetooth/BluetoothPan;->isTetheringOn()Z`](https://android.googlesource.com/platform/prebuilts/runtime/+/94fec32/appcompat/hiddenapi-light-greylist.txt#1498)
* (since API 24) [`Landroid/net/ConnectivityManager;->getLastTetherError(Ljava/lang/String;)I`](https://android.googlesource.com/platform/prebuilts/runtime/+/94fec32/appcompat/hiddenapi-light-greylist.txt#3783)
* (deprecated since API 26) [`Landroid/net/wifi/WifiManager;->setWifiApEnabled(Landroid/net/wifi/WifiConfiguration;Z)Z`](https://android.googlesource.com/platform/prebuilts/runtime/+/94fec32/appcompat/hiddenapi-dark-greylist.txt#4378)
* [`Landroid/net/wifi/p2p/WifiP2pGroup;->getNetworkId()I`](https://android.googlesource.com/platform/prebuilts/runtime/+/94fec32/appcompat/hiddenapi-light-greylist.txt#4399)
* [`Landroid/net/wifi/p2p/WifiP2pGroupList;->getGroupList()Ljava/util/Collection;`](https://android.googlesource.com/platform/prebuilts/runtime/+/94fec32/appcompat/hiddenapi-light-greylist.txt#4405)
* [`Landroid/net/wifi/p2p/WifiP2pManager;->deletePersistentGroup(Landroid/net/wifi/p2p/WifiP2pManager$Channel;ILandroid/net/wifi/p2p/WifiP2pManager$ActionListener;)V`](https://android.googlesource.com/platform/prebuilts/runtime/+/94fec32/appcompat/hiddenapi-light-greylist.txt#4411)
* [`Landroid/net/wifi/p2p/WifiP2pManager;->requestPersistentGroupInfo(Landroid/net/wifi/p2p/WifiP2pManager$Channel;Landroid/net/wifi/p2p/WifiP2pManager$PersistentGroupInfoListener;)V`](https://android.googlesource.com/platform/prebuilts/runtime/+/94fec32/appcompat/hiddenapi-light-greylist.txt#4412)
* [`Landroid/net/wifi/p2p/WifiP2pManager;->setWifiP2pChannels(Landroid/net/wifi/p2p/WifiP2pManager$Channel;IILandroid/net/wifi/p2p/WifiP2pManager$ActionListener;)V`](https://android.googlesource.com/platform/prebuilts/runtime/+/94fec32/appcompat/hiddenapi-light-greylist.txt#4416)
* [`Landroid/net/wifi/p2p/WifiP2pManager;->startWps(Landroid/net/wifi/p2p/WifiP2pManager$Channel;Landroid/net/wifi/WpsInfo;Landroid/net/wifi/p2p/WifiP2pManager$ActionListener;)V`](https://android.googlesource.com/platform/prebuilts/runtime/+/94fec32/appcompat/hiddenapi-light-greylist.txt#4417)
* [`Ljava/net/InetAddress;->parseNumericAddress(Ljava/lang/String;)Ljava/net/InetAddress;`](https://android.googlesource.com/platform/prebuilts/runtime/+/94fec32/appcompat/hiddenapi-light-greylist.txt#9800)

Unlisted private API:

* (since API 24) `Landroid/net/ConnectivityManager;->startTethering(IZLandroid/net/ConnectivityManager$OnStartTetheringCallback;Landroid/os/Handler;)V`
* (since API 24) `Landroid/net/ConnectivityManager;->stopTethering(I)V`

Private system configurations:

* `@android:array/config_tether_usb_regexs`
* `@android:array/config_tether_wifi_regexs`
* `@android:array/config_tether_wimax_regexs`
* `@android:array/config_tether_bluetooth_regexs`

Other:

* (since API 27) `android.provider.Settings.Global.TETHER_OFFLOAD_DISABLED` is assumed to be
  `"tether_offload_disabled"`.
* (since API 27) `com.android.server.connectivity.tethering.OffloadHardwareInterface.DEFAULT_TETHER_OFFLOAD_DISABLED`
  is assumed to be 0.
* Activity `com.android.settings/.Settings$TetherSettingsActivity` is assumed to be exported;
* Several constants in `ConnectivityManager` is assumed to be defined as in `TetheringManager.kt`;
* `android.net.conn.TETHER_STATE_CHANGED` and `android.net.wifi.p2p.PERSISTENT_GROUPS_CHANGED` are assumed to be
  sticky broadcasts.

For `ip rule` priorities, `RULE_PRIORITY_SECURE_VPN` and `RULE_PRIORITY_TETHERING` is assumed to be 12000 and 18000
respectively; `RULE_PRIORITY_DEFAULT_NETWORK` is assumed to be 22000 (or at least > 18000) for API 27-.
DHCP server like `dnsmasq` is assumed to run and send DHCP packets as root.

Undocumented system binaries are all bundled and executable:

* Since API 24: `iptables-save`;
* `echo`;
* `ip` (`link monitor neigh rule` with proper output format);
* `iptables` (with correct version corresponding to API level, `-nvx -L <chain>` with proper output format);
* `su`.

If some of these are unavailable, you can alternatively install a recent version (v1.28.1 or higher) of Busybox.

Wi-Fi driver `wpa_supplicant`:

* P2P configuration file is assumed to be saved to `/data/misc/wifi/p2p_supplicant.conf` or
  `/data/vendor/wifi/wpa/p2p_supplicant.conf` for API 28+ and have reasonable format;
* Android system is expected to restart `wpa_supplicant` after it crashes.
