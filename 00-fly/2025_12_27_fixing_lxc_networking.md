## Fixing LXC Networking

Usually after running `incus launch image:...` the built image is doesn't get a IP4, that is usually caused by the host firewall blocking the DHCP to the bridge interface.

All my setups rely on nftables (with or without iptables-nft) and I prefer to use the nft interface to manage the firewall rules. So check in particular the forward rules for the different tables (inet and ip) the incus interface should be allow and more if any of those chains have a rule that drops all the traffic `policy drop`.

If that' the case just add the rules to allow the traffix to the interface (input and output traffix as it is a bridge interface)
