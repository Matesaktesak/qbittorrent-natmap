# qBittorrent-NatMap 

This container periodicity runs a script requesting a port forward (via NAT-PMP) from the VPN provider and changes the listening port in preexisting qBittorrent container accordingly.

This solution is currently being tested with ProtonVPN using Wireguard and qBitTorrent container from Linuxserver.io.

## What made me do this?

The need to improve the seeding/upload performance and not finding any work done for this scenario (qBittorrent using Wireguard on UnRAID), but finding [this post on reddit](https://old.reddit.com/r/ProtonVPN/comments/10owypt/successful_port_forward_on_debian_wdietpi_using/) by u/TennesseeTater for Deluge made me try and do something similar. His post is also referenced in the [ProtonVPN Guide][1].

## Why not use a scheduled script on host unRAID?

Well, as far as I could find, unRAID doesn't include **natpmpc**, the NAT-PMP client used to request the *port forward* as per the instructions for [manual mapping][1] on ProtonVPN. And you shouldn't install software directly on the host OS.

## What does the script do/modify?

So far:

* Evaluates the required variables for execution (for now, if they're set) and if **docker.sock** was mapped from the host into the container
* If the above succeeds:
    * Get the VPN public IP
    * Grab the SessionID cookie from qBittorrent
    * Grab the current listen port from qBittorrent
* After the configured checks pass, a function to request and verify the port mapping starts
    * Using *natpmpc* a port mapping request is made to the address defined in `VPN_GATEWAY` for udp and tcp
    * Comparison is made between the current configured port in qBittorrent and the currently active mapped port from the VPN
    * If a condition of different port is found:
        * The new port is configured in qBittorrent, along with that it's also disabled the random port setting and UPnP mapping from the torrent client
        * A couple of commands are executed to add/remove iptables rules regarding the previous active and new active mapped port on the VPN container

These actions are performed continuously (in a loop, every 60 seconds (default).

## Configurable variables:

* QBITTORRENT_SERVER (Defaults to **qbittorrent**)
* QBITTORRENT_PORT (Defaults to **8080**)
* QBITTORRENT_USER (Defaults to **admin**)
* QBITTORRENT_PASS (Defaults to **adminadmin**)
* VPN_GATEWAY (Defaults to **empty**)
    * If not set the script will try and find it
    * The value for this variable will be the `VPN_IF_NAME` (default: wg0) gateway address, not the `VPN_ENDPOINT_IP` from Wireguard.
    * For ProtonVPN using Wireguard this would be set to **10.2.0.1**
* VPN_IF_NAME (Defaults to **wg0**)
* CHECK_INTERVAL (Defaults to **60s**)
* NAT_LEASE_LIFETIME (Defaults to **60s**)
    * Ideally both `CHECK_INTERVAL` and `NAT_LEASE_LIFETIME` should be set equal or the check interval lower than the lease lifetime, but never above.

[1]: https://protonvpn.com/support/port-forwarding-manual-setup/
