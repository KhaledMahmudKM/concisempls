# Appendix: Command Reference

The commands used in the labs, grouped by task. FRR commands run inside `vtysh` (`docker exec -it clab-<lab>-<node> vtysh`). Linux commands run against a router's network namespace from the host (`sudo ip netns exec clab-<lab>-<node> <command>`).

## Lab lifecycle

| Command | Purpose |
|---|---|
| `sudo modprobe mpls_router mpls_iptunnel` | Load the MPLS modules on the host (once per boot) |
| `ls /proc/sys/net/mpls` | Check that the kernel supports MPLS |
| `sudo containerlab deploy -t <lab>.clab.yml` | Start a lab |
| `sudo containerlab inspect -t <lab>.clab.yml` | List nodes and management addresses |
| `sudo containerlab destroy -t <lab>.clab.yml --cleanup` | Stop a lab and remove its files |
| `docker exec -it clab-<lab>-<node> vtysh` | FRR shell on a router |
| `docker exec -it clab-<lab>-<node> sh` | Linux shell on a router or host |

## Linux MPLS data plane

| Command | Purpose |
|---|---|
| `sysctl net.mpls.platform_labels` | Size of the label table (0 disables MPLS) |
| `sysctl -w net.mpls.conf.eth1.input=1` | Accept labelled packets on eth1 |
| `ip -f mpls route show` | The kernel's label forwarding table |
| `ip route show 10.0.0.2` | An IP route, showing `encap mpls` labels to push |
| `ip route show vrf RED` | A VRF's routing table |
| `ip link add RED type vrf table 10` | Create a VRF |
| `ip link set eth3 master RED` | Put an interface into a VRF |
| `ip -6 route show \| grep seg6` | SRv6 encapsulation and local SID entries |
| `tcpdump -nni eth1 mpls` | Capture labelled packets |

## IS-IS

| Command | Purpose |
|---|---|
| `show isis neighbor` | Adjacencies |
| `show isis database detail` | The link-state database, including SR sub-TLVs |
| `show isis route` | IS-IS routes with next hops and labels |
| `show ip route 10.0.0.2/32` | The installed route, with its label |

## LDP

| Command | Purpose |
|---|---|
| `show mpls ldp discovery` | Hellos heard per interface |
| `show mpls ldp neighbor` | LDP sessions and their state |
| `show mpls ldp binding` | Local and remote labels per FEC, and which are in use |
| `show mpls ldp interface` | Interfaces with LDP enabled |
| `show mpls table` | The label forwarding table, from all protocols |

## BGP and L3VPN

| Command | Purpose |
|---|---|
| `show bgp summary` | All BGP sessions and address families |
| `show bgp ipv4 unicast` | IPv4 routes (BGP-free core lab) |
| `show bgp ipv4 vpn` | VPNv4 routes, grouped by RD, with labels and RTs |
| `show bgp vrf RED ipv4 unicast` | A VRF's BGP table after import |
| `show ip route vrf RED` | A VRF's routing table, with transport and VPN labels |
| `clear bgp ipv4 unicast <peer> soft in` | Re-apply inbound policy |

## Segment Routing and fast reroute

| Command | Purpose |
|---|---|
| `show isis segment-routing node` | SR capabilities and node SIDs of every router |
| `show isis fast-reroute summary` | Protection coverage per priority |
| `show isis route backup` | Repair paths computed by LFA or TI-LFA |
| `show sr-te policy detail` | SR policies, candidate paths, active segment list |
| `show segment-routing srv6 locator` | SRv6 locators and their state |
| `show isis segment-routing srv6 node` | SRv6 locators learned from every router |

## Configuration snippets

**LDP** (Chapter 4):

```text
mpls ldp
 router-id 10.0.0.11
 address-family ipv4
  discovery transport-address 10.0.0.11
  label local allocate host-routes
  interface eth1
  exit
 exit-address-family
exit
```

**L3VPN VRF** (Chapter 6):

```text
router bgp 65000 vrf RED
 address-family ipv4 unicast
  redistribute static
  label vpn export auto
  rd vpn export 10.0.0.1:10
  rt vpn both 65000:10
  export vpn
  import vpn
 exit-address-family
exit
```

**SR-MPLS with TI-LFA** (Chapters 9 and 10):

```text
router isis CORE
 segment-routing on
 segment-routing global-block 16000 23999
 segment-routing prefix 10.0.0.1/32 index 1
exit
interface eth1
 isis fast-reroute ti-lfa
exit
```

**SR policy** (Chapter 11):

```text
segment-routing
 traffic-eng
  segment-list VIA-P3
   index 10 mpls label 16013
   index 20 mpls label 16002
  exit
  policy color 10 endpoint 10.0.0.2
   candidate-path preference 100 name VIA-P3 explicit segment-list VIA-P3
  exit
 exit
exit
```

**SRv6 L3VPN** (Chapter 12): see the full configuration in that chapter.
