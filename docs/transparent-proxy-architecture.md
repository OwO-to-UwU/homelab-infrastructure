# Transparent traffic processing on Proxybox

Proxybox is a dedicated Debian Linux gateway for selected LAN flows. RouterOS sends eligible traffic to it over a transit link; nftables and B4 handle classification and interception, while sing-box provides outbound proxy/tunnel connectivity.

## Packet-processing stages

| Component | Role |
| --- | --- |
| RouterOS `via-proxybox` | Steers selected LAN traffic to the Debian gateway |
| `proxybox_guard` | Marks local and special-use IPv4 destinations for bypass |
| `b4_mangle` | NFQUEUE classification for selected DNS and HTTPS traffic |
| `b4_route` | TPROXY interception of matching TCP and UDP flows |
| `ip rule` | Directs marked packets into a local-delivery routing table (`local default dev lo`) |
| B4 | Owns the transparent TCP/UDP listening sockets |
| sing-box | Runs the configured outbound connections and local SOCKS listeners |
| `proxybox_transit` | Applies SNAT on selected transit-to-LAN forwarded flows |

The ordering of nftables rules matters: early bypass marking prevents special/private destinations from being treated like ordinary proxied Internet traffic. NFQUEUE and TPROXY have different jobs — packet classification versus redirection to transparent sockets.

## Routing identities

The gateway has separate IPv4 addresses for LAN management and different egress roles. RouterOS uses matching source-based policy rules, so forwarded traffic can take the intended ISP or public-IPv4 path after processing. The transit interface is separate from the main LAN interface.

## Service startup ordering

Secondary LAN addresses are configured by a systemd oneshot unit. A second oneshot checks that the expected addresses and default route are present, and the WireGuard, B4 and sing-box units require that readiness gate. See the [systemd configuration examples](../examples/systemd/proxybox/README.md).

For router-side tables, internal DNS and external access, see [Network architecture](network-architecture.md).
