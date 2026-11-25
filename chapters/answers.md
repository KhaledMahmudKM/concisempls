# Answers to Chapter Questions

Answers and short explanations for the **Check your understanding** questions at the end of each chapter. Try the questions first; each section links back to them.

## Chapter 1: Why MPLS

**1. A. A short label chosen at the network edge**

The ingress picks the label once; core routers just look up the label.

**2. C. LSR**

Label Edge Routers push and pop labels at the edges; Label Switching Routers swap them.

**3. B. It is one-directional; return traffic needs its own LSP**

Two-way traffic uses two LSPs.

**4. D. A set of packets forwarded the same way, and so given the same label**

For example, all packets heading for one egress PE.

**5. B. A label acts as a tunnel, enabling a BGP-free core, VPNs, traffic engineering and fast reroute**

The original motivation, faster lookups, no longer matters.

**6. D. PE**

P routers just switch labels; CE routers belong to the customer.

**7. A. CE**

The CE peers with the provider's PE using normal IP routing.

**8. C. DHCP**

IS-IS or OSPF with Segment Routing can also carry labels.

[Back to the questions](01-why-mpls.md#check-your-understanding)

## Chapter 2: Labels and Forwarding

**1. C. 20 bits**

The whole entry is 32 bits: 20-bit label, 3-bit TC, 1-bit S and 8-bit TTL.

**2. B. The bottom of the stack**

It is set on the last label before the payload.

**3. C. 0x8847**

0x0800 is IPv4, 0x86DD IPv6, 0x8100 a VLAN tag.

**4. A. Each router chooses its own labels, which mean something only to that router**

The same LSP can use a different label on each hop.

**5. C. The LFIB**

The FIB maps prefixes to labels to push at the ingress.

**6. D. 3, implicit null**

With PHP, the router before the egress pops the last label.

**7. B. To keep a label, with its TC bits, all the way to the egress**

The egress pops it itself and can still read the QoS marking.

**8. C. net.mpls.conf.<if>.input=1**

net.mpls.platform_labels must also be large enough for the labels in use.

[Back to the questions](02-labels.md#check-your-understanding)

## Chapter 3: The Lab

**1. A. FRR containers under containerlab**

FRR provides the control plane; the Linux kernel forwards the labelled packets.

**2. C. On the host, since containers cannot load kernel modules**

All containers share the host's kernel.

**3. D. The Linux kernel**

FRR programs routes and labels into the kernel, which does the forwarding.

**4. B. The highest label number the kernel will accept, the size of its label table**

If it is too small, labels are rejected.

**5. C. IS-IS level 2**

pe1, pe2, p1, p2 and p3 form the core.

**6. A. On purpose, to show that VPNs keep overlapping customer address spaces apart**

Chapter 6 shows how VRFs and RDs handle the overlap.

**7. A. sysctls, binds and exec**

links define the cabling between nodes.

**8. C. The SRv6 lab in Chapter 12**

Check the running kernel with uname -r.

[Back to the questions](03-lab.md#check-your-understanding)

## Chapter 4: LDP

**1. B. Each IGP prefix, on each router**

LDP builds no paths of its own: its LSPs follow the IGP.

**2. C. UDP 646 Hellos**

Labels are then exchanged over one TCP 646 session per neighbour.

**3. A. Loopbacks**

The LSR ID and transport address are usually the loopback.

**4. A. Upstream, from the egress toward the ingress**

Each router tells its upstream neighbours which label to use toward a prefix.

**5. C. Downstream unsolicited with liberal retention**

Liberal retention keeps labels from non-best neighbours, ready for faster reconvergence.

**6. B. Restrict label allocation to host routes, such as loopbacks**

Only PE loopbacks really need LSPs.

**7. D. A link carrying IGP traffic before LDP has labels for it, dropping labelled traffic**

The IGP advertises the link with a high metric until LDP is ready.

**8. B. The LDP session, over another path, so labels are ready when the link returns**

It uses targeted Hellos to maintain the session.

[Back to the questions](04-ldp.md#check-your-understanding)

## Chapter 5: The BGP-Free Core

**1. A. Only the PEs**

P routers carry routes and labels for PE loopbacks only.

**2. C. Recursive resolution: the BGP next hop is a PE loopback, whose IGP route carries an LDP label**

The core switches on the label for the egress PE's loopback.

**3. B. So every BGP next hop is a PE loopback with an LSP**

External next hops have no labels in the core.

**4. D. It is dropped, since P routers have no route for the destination**

Keep LSPs healthy and synchronised with the IGP.

**5. B. Carries IPv6 over an IPv4 MPLS core, using a second label**

6PE is RFC 4798; 6VPE does the same for VPNs.

**6. D. LSPs that extend across separate IGP domains**

It is the basis of seamless MPLS.

**7. A. An LSP between their loopbacks**

The loopback must have an LSP from every PE to every PE.

**8. C. Nothing: it has no route at all for them**

That is why it must forward on labels.

[Back to the questions](05-bgp-free-core.md#check-your-understanding)

## Chapter 6: BGP/MPLS Layer 3 VPNs

**1. D. A VRF**

VRF-lite is the same idea without MPLS.

**2. B. Makes overlapping customer prefixes unique in BGP**

The RD itself expresses no policy.

**3. D. Decides which VRFs import a route**

A VRF exports routes with RTs and imports routes with matching RTs.

**4. B. AFI 1, SAFI 128**

VPNv6 is the IPv6 equivalent.

**5. C. Two: a transport label and a VPN label**

The transport label reaches the egress PE; the VPN label selects the VRF there.

**6. D. Which VRF, and so which customer, the packet belongs to**

The egress PE pops it and routes the packet in that VRF.

**7. B. Nothing: they just switch on the transport label**

That keeps the core simple and scalable.

**8. A. RFC 4364**

RFC 3031 is the MPLS architecture, RFC 5036 LDP, RFC 8402 Segment Routing.

[Back to the questions](06-l3vpn.md#check-your-understanding)

## Chapter 7: L3VPN Design

**1. A. A full mesh of iBGP sessions between PEs**

Each PE peers only with the route reflectors.

**2. C. Sends each PE only the VPN routes it imports**

PEs advertise which RTs they want, and the RR filters accordingly.

**3. D. Through the hub, because spokes import only the hub's routes**

Import and export RTs build the topology.

**4. B. They must not overlap with the customers' prefixes that import them**

Every customer importing them must be able to route to them unambiguously.

**5. C. eBGP**

Static routing, OSPF and IS-IS are also used.

**6. A. AS override**

The PE replaces the customer's ASN with its own; allowas-in on the CE is the alternative.

**7. A. Option A**

It is simple and isolating, but scales poorly.

**8. C. Option C**

VPNv4 routes then flow between route reflectors over multihop eBGP; it needs trust between providers.

[Back to the questions](07-l3vpn-design.md#check-your-understanding)

## Chapter 8: Layer 2 VPNs

**1. B. A point-to-point link carrying the customer's frames**

It carries a transport label, a PW label, an optional control word, then the frame.

**2. C. Stopping core routers from mistaking the Ethernet payload for an IP packet when load balancing**

It also carries a sequence number.

**3. A. VPWS (E-Line)**

VPLS uses a full mesh of pseudowires.

**4. A. In the data plane, by learning from received frames**

It learns, forwards and floods like a switch.

**5. C. So a frame received from one pseudowire is not sent out another, preventing loops**

The full mesh guarantees every PE receives the frame directly.

**6. B. It advertises them in BGP, in the control plane**

That enables all-active multihoming, fast convergence and less flooding.

**7. D. The Ethernet segment identifier (ESI)**

PEs sharing an ESI know they serve the same site.

**8. B. With EVPN-VXLAN in FRR, since Linux has no MPLS data plane for these services**

EVPN runs over MPLS or VXLAN; the control plane is the same.

[Back to the questions](08-l2vpn.md#check-your-understanding)

## Chapter 9: Segment Routing with MPLS

**1. A. As a list of segments carried in the packet**

On MPLS, each segment ID is a label.

**2. C. Prefix (node) SID**

Adjacency SIDs are local and pin one link.

**3. B. SRGB base + index**

With an SRGB starting at 16000, index 2 is label 16002.

**4. D. So the label for a node is the same everywhere, which makes label stacks readable**

The standard still works with different SRGBs, but it is harder to read.

**5. B. IS-IS or OSPF**

No LDP is needed.

**6. D. Go to p3 by the shortest path, then to pe2 by the shortest path**

Adding an adjacency SID would pin one particular link.

**7. A. One protocol fewer, global labels, TE without signalling, and full fast reroute coverage**

Services such as L3VPN work unchanged over SR.

**8. C. Through stitching and a mapping server**

The mapping server advertises SIDs on behalf of LDP-only routers.

[Back to the questions](09-sr-mpls.md#check-your-understanding)

## Chapter 10: Fast Reroute

**1. D. Pre-installs backup paths so the PLR can switch within tens of milliseconds**

The IGP still converges normally afterwards.

**2. B. The point of local repair: the router next to the failure that switches to the backup**

It detects the failure and repairs locally.

**3. D. When dist(N, D) < dist(N, S) + dist(S, D)**

N must not route back through S.

**4. A. Its coverage depends on the topology; some failures have no LFA**

Remote LFA and TI-LFA improve coverage.

**5. C. A node in both S's P space and E's Q space: a safe tunnel endpoint**

S tunnels traffic to it with an LDP label learned over targeted LDP.

**6. D. The post-convergence path, encoded as a segment list**

Traffic goes where the network will settle anyway.

**7. B. Any single link, node or SRLG failure**

An SRLG is a group of links that share a risk, such as a fibre duct.

**8. C. isis fast-reroute ti-lfa**

Enable it on each core interface.

[Back to the questions](10-fast-reroute.md#check-your-understanding)

## Chapter 11: Traffic Engineering

**1. A. Shortest-path routing sends all traffic on the same paths, which can congest them while others sit idle**

TE places traffic on chosen paths.

**2. C. With constrained SPF (CSPF) over the TE database**

Constraints include bandwidth and link attributes.

**3. D. PATH and RESV**

PATH carries the Explicit Route Object; RESV returns labels and reservations.

**4. B. Per-tunnel state in every core router on the path**

SR policies keep no per-path state in the core.

**5. C. Head end, color and endpoint**

Each policy has prioritised candidate paths, explicit or dynamic.

**6. A. The route's color and next hop**

A route with color 100 and next hop pe2 uses the policy with color 100 to pe2.

**7. A. When a path needs a global view of the network, delivered to head ends over PCEP**

It can learn the topology through BGP-LS.

**8. C. pathd**

pathd can also talk PCEP to a controller.

[Back to the questions](11-traffic-engineering.md#check-your-understanding)

## Chapter 12: SRv6 and the Road Ahead

**1. B. An IPv6 address**

The destination address is the active segment.

**2. C. A locator, routed by the IGP, and a function**

The function names the behaviour, such as End, End.X or End.DT4.

**3. A. In a Segment Routing Header (SRH)**

Segments left points to the active segment.

**4. A. Up to six segments packed into one IPv6 address**

With the F3216 format, a 32-bit block is followed by 16-bit uSIDs.

**5. C. End.DT4**

End.DT6 does the same for IPv6.

**6. B. An End.DT4/DT6 SID per VRF, advertised in BGP**

The BGP VPN machinery is otherwise the same.

**7. D. sid vpn per-vrf export auto**

label vpn export auto is the MPLS equivalent.

**8. B. SR-MPLS**

SRv6 suits new IPv6 networks; the services stay the same.

[Back to the questions](12-srv6.md#check-your-understanding)
