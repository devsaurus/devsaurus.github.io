# Filtering web sites using firewall IP sets

## Introduction

This article shows a practical approach for how to filter web sites at your router. The approach combines two mechanisms:

1. DNS name resolution to obtain IP addresses
1. Firewall [IP sets](https://en.wikipedia.org/wiki/Netfilter#ipset) to filter based on IP addresses

This allows to filter for domain names that resolve dynamically to different IP addresses.

## Concept

The concept is to instruct the DNS name resolver to collect IP addresses that were obtained for certain domain names in IP sets. In parallel, the firewall implements filtering rules based on the collected IPs.

Outline of the DNS resolver side:
1. Client requests name resolution for example.com
1. The DNS resolver matches domain against a list of domains
1. If domain matches then the resolved IP addresses is put into an IP set
1. The resolved IP address is returned to the client

Outline of the firewall side:
1. Client sends packets to example.com using the resolved IP address
1. The firewall matches the destination IP against the members of the IP set
1. If the desintation IP matches then the packet is rejected

## Implementation

OpenWRT is used to implement the concept. The following chapters are inspired by [DNS-based firewall with IP sets](https://openwrt.org/docs/guide-user/firewall/fw3_configurations/dns_ipset).

### Pre-conditions

The following packages have to be installed on the router:

```
opkg update

# remove the pre-installed basic dnsmasq
opkg remove dnsmasq

opkg install dnsmasq-full ipset
```

### Firewall setup

#### IP sets

A pair of IP sets is created in `/etc/config/firewall`, one for IPv4 and one for IPv6:

```sh
# Configure IP sets
uci -q delete firewall.filter
uci set firewall.filter="ipset"
uci set firewall.filter.name="filter"
uci set firewall.filter.family="ipv4"
uci set firewall.filter.storage="hash"
uci set firewall.filter.match="ip"
uci -q delete firewall.filter6
uci set firewall.filter6="ipset"
uci set firewall.filter6.name="filter6"
uci set firewall.filter6.family="ipv6"
uci set firewall.filter6.storage="hash"
uci set firewall.filter6.match="ip"

uci commit firewall
/etc/init.d/firewall restart
```

Run `ipset list` to see the effect. Note that they don't contain any members yet.

#### Filter rules

A pair of filter rules is created in `/etc/config/firewall`, again one for IPv4 and one for IPv6:

```sh
# Filter LAN client traffic with IP sets
uci -q delete firewall.filter_fwd
uci set firewall.filter_fwd="rule"
uci set firewall.filter_fwd.name="Filter-IPset-DNS-Forward"
uci set firewall.filter_fwd.src="lan"
uci set firewall.filter_fwd.dest="wan"
uci set firewall.filter_fwd.ipset="filter dest"
uci set firewall.filter_fwd.family="ipv4"
uci set firewall.filter_fwd.proto="all"
uci set firewall.filter_fwd.target="REJECT"
uci -q delete firewall.filter6_fwd
uci set firewall.filter6_fwd="rule"
uci set firewall.filter6_fwd.name="Filter6-IPset-DNS-Forward"
uci set firewall.filter6_fwd.src="lan"
uci set firewall.filter6_fwd.dest="wan"
uci set firewall.filter6_fwd.ipset="filter6 dest"
uci set firewall.filter6_fwd.family="ipv6"
uci set firewall.filter6_fwd.proto="all"
uci set firewall.filter6_fwd.target="REJECT"

uci commit firewall
/etc/init.d/firewall restart
```
#### Extras

See [DNS-based firewall with IP sets -> Extras](https://openwrt.org/docs/guide-user/firewall/fw3_configurations/dns_ipset#extras) for further tweaking of the firewall rules.


### dnsmasq setup

The domain names that should feed into the IP sets are added in `/etc/config/dhcp`:

```sh
uci add_list dhcp.@dnsmasq[0].ipset='/example.com/filter,filter6'
uci add_list dhcp.@dnsmasq[0].ipset='/example.org/filter,filter6'

uci commit dhcp
/etc/init.d/dnsmasq restart
```

Note that each domain name feeds into both IP sets for IPv4 and IPv6.


## Conclusion

With the setup shown above, traffic to example.com and example.org is blocked even if the domain names resolve dynamically to different IP addresses.
