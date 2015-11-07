The NE2000 patch by Peter Grahen brings the Bochs NE2000 network card 
emulation code to dosbox. 

This updated patch will cleanly patch newer dosbox versions; I used dosbox SVN 
r3950 as a base.

The driver (most of the time) needs root access in order to talk to the network 
interface, which can be frustrating.

I've added some code to Peter's patch, which wraps libpcap calls around setuid.
That way you can run dosbox as your own user, while the NE2000 code runs as
the owner of the dosbox binary (i.e root). 

This is NOT a very safe thing to do, but it's safer than running the entire
emulator as root, as the original patch required.

--- Sidenote ---
A safer way to gain raw network access is to enable Linux System Capabilities 
and set the dosbox binary with the CAP_NET_RAW and CAP_NET_ADMIN
capabilities (`setcap cap_net_raw,cap_net_admin=eip /path/to/dosbox`), 
but this requires Linux (I don't think it'll work on FreeBSD et al), and you
need to have the capabilities enabled in your kernel (i.e you must have
`file_caps=1` in your kernel parameters). If you're interested in this 
approach, have a look at:
http://peternixon.net/news/2012/01/28/configure-tcpdump-work-non-root-user-opensuse-using-file-system-capabilities/
Currently, the NE2000 code doesn't check for these capabilities. So even if
you jump through these hoops, the patch will try to gain suid privs.
--- Sidenote ---

So, once built, make the dosbox binary suid.
As root,
	chown root /path/to/dosbox
   	chmod u+s /path/to/dosbox

For help on configuring DOSBox to utilize the new features, please
have a look at the daughter repository:
https://github.com/gammy/dos_net_ne2000

To enable aggressive verbosity to the NE2000 driver, set (uncomment)
NE2K_DEBUGMSG in ne2000.h.

Please note that I've lazily disabled the promiscuous mode check in 
be_ne2k::rx_frame: For some reason on my machine, a condition in this method
determines that the card is not in promiscuous mode (when it really is), 
and as a result `mcast_index` returns an invalid index, resulting in an
out-of-bounds error, crashing dosbox (and the debugger).
I'm not sure why this check fails. Either way, the card *ought* already to be
in promiscuous mode, as libpcap sets the promiscuous state during driver
initialization. I.e it works just fine without the re-check upon receiving a
packet.
