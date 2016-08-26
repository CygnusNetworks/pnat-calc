# iptables Port NAT Tool

This small tool creates iptables rules to map a internal subnet onto a smaller external subnet by using fixed ports numbers for TCP and UDP for the external IPs for specific internal ips.

Example:

```
./pnat-calc -i 192.168.0.0/24 -e 128.128.1.0/32
```

will map the:
  - internal IP 192.168.0.0 to 128.128.1.0 with TCP/UDP Port range 1024-1275
  - internal IP 192.168.0.1 to 128.128.1.0 with TCP/UDP Port range 1276-1527
  - ...

Port ranges are calculated automatically depending on size of the networks or can be specified by using -p parameter. Minmum and Maximum port numbers can be specified by using -m (default 1024) or -M (default 65535) parameters respectively. IPTables chains are split up by external IP.

You will need to add a NAT rule to jump to the generated chains like: 

```
iptables -t nat -A POSTROUTING -s 192.168.0.0/24 -o eth0 -j PNATCALC
```

## Requirements:
  - Python 3.3 or higher
  
## Usage

```
./pnat-calc -h                                                                                                                                                                             master↑2| 
usage: pnat-calc [-h] [-c CHAIN] [-b IPTABLES] -i INTERNAL -e EXTERNAL
                 [-m MIN_PORT] [-M MAX_PORT] [-p PORTS_PER_IP]

optional arguments:
  -h, --help            show this help message and exit
  -c CHAIN, --chain CHAIN
                        chain name
  -b IPTABLES, --iptables IPTABLES
                        iptables path or binary name
  -i INTERNAL, --internal INTERNAL
                        whitespace separated list of internal
                        networks/adresses (e.g. "192.168.100.0/24
                        192.168.101.128/25")
  -e EXTERNAL, --external EXTERNAL
                        whitespace separated list of external
                        networks/adresses (e.g. "198.51.100.0/24 203.0.113.1")
  -m MIN_PORT, --min-port MIN_PORT
                        minimum source port (between 1024 and 65535, default:
                        1024)
  -M MAX_PORT, --max-port MAX_PORT
                        maximum source port (between 1024 and 65535, default:
                        65535)
  -p PORTS_PER_IP, --ports-per-ip PORTS_PER_IP
                        number of ports per internal ip address (between 1 and
                        64512, default: auto)
```

## Example Output

```./pnat-calc -i 192.168.0.0/24 -e 128.128.1.0/32```

will generate:

```
iptables -t nat -N PNATCALC_128.128.1.0
iptables -t nat -F PNATCALC_128.128.1.0
iptables -t nat -A PNATCALC_128.128.1.0 --source 192.168.0.0 -p tcp -j SNAT --to-source 128.128.1.0:1024-1275
iptables -t nat -A PNATCALC_128.128.1.0 --source 192.168.0.0 -p udp -j SNAT --to-source 128.128.1.0:1024-1275
iptables -t nat -A PNATCALC_128.128.1.0 --source 192.168.0.1 -p tcp -j SNAT --to-source 128.128.1.0:1276-1527
iptables -t nat -A PNATCALC_128.128.1.0 --source 192.168.0.1 -p udp -j SNAT --to-source 128.128.1.0:1276-1527
iptables -t nat -A PNATCALC_128.128.1.0 --source 192.168.0.2 -p tcp -j SNAT --to-source 128.128.1.0:1528-1779
iptables -t nat -A PNATCALC_128.128.1.0 --source 192.168.0.2 -p udp -j SNAT --to-source 128.128.1.0:1528-1779
...
iptables -t nat -A PNATCALC_128.128.1.0 --source 192.168.0.255 -p tcp -j SNAT --to-source 128.128.1.0:65284-65535
iptables -t nat -A PNATCALC_128.128.1.0 --source 192.168.0.255 -p udp -j SNAT --to-source 128.128.1.0:65284-65535
iptables -t nat -A PNATCALC_128.128.1.0 -j SNAT --to-source 128.128.1.0
iptables -t nat -A PNATCALC -m iprange --src-range 192.168.0.0-192.168.0.255 -j PNATCALC_128.128.1.0
```
