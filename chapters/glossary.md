# Glossary

**6PE.** Carrying IPv6 routes and traffic across an IPv4 MPLS core using iBGP with labels (RFC 4798). **6VPE** does the same for IPv6 VPN routes.

**Adjacency SID.** A segment that is local to one router and means "send out of this link".

**Attachment circuit (AC).** The port or VLAN connecting a customer to a PE in a layer 2 VPN.

**Binding SID (BSID).** A label or SRv6 SID that stands for a whole SR policy, so other routers can steer traffic into it.

**BGP labeled unicast (BGP-LU).** BGP advertising IPv4 or IPv6 routes with labels (RFC 8277), used to extend LSPs across IGP domains.

**Bottom of stack (S bit).** The bit set in the last label stack entry.

**CE (customer edge).** The customer's router facing the provider. It runs plain IP.

**Color.** In SR policies, a number standing for an intent such as low latency; BGP routes carry it as an extended community.

**Control word.** An optional 4-byte field after the pseudowire label.

**CSPF.** Constrained shortest path first: SPF after pruning links that do not meet constraints such as bandwidth.

**Downstream unsolicited (DU).** LDP mode in which routers advertise labels without being asked.

**Egress, ingress.** The routers where an LSP ends and begins.

**Ethernet segment (ES), ESI.** In EVPN, the set of links connecting a multihomed CE to several PEs, and its identifier.

**EVPN.** A BGP control plane for layer 2 (and layer 3) VPNs that advertises MAC addresses (RFC 7432).

**Explicit null.** Reserved label 0 (IPv4) or 2 (IPv6). Asking for it keeps a label on the last hop instead of PHP.

**Extended community.** A BGP path attribute used for route targets, colors, and Site of Origin.

**FEC (forwarding equivalence class).** A group of packets forwarded the same way, such as all packets to one egress router.

**Implicit null.** Reserved label 3. Advertising it asks the upstream router to pop (PHP); it is never sent on the wire.

**Index.** A prefix SID's position within the SRGB. Label = SRGB base + index.

**Inter-AS options A, B, C.** Three ways of joining VPNs across providers (RFC 4364 section 10).

**IS-IS.** A link-state IGP widely used in provider cores, and the IGP of the labs.

**Label.** A 20-bit value in a label stack entry that a router looks up to forward a packet.

**Label stack entry.** The 32-bit MPLS header: label, TC, S bit, TTL.

**LDP.** Label Distribution Protocol (RFC 5036): assigns labels to IGP prefixes hop by hop.

**LDP-IGP synchronisation.** Keeping a link at maximum IGP metric until LDP is ready on it (RFC 5443).

**LER (label edge router).** A router at the edge of an MPLS network that pushes or pops labels.

**LFA (loop-free alternate).** A neighbour that can take traffic for a destination without sending it back.

**LFIB.** The label forwarding table: incoming label to action and next hop.

**Liberal retention.** Keeping labels from all neighbours, not just the current next hop.

**Locator.** The part of an SRv6 SID that routes to the owning node.

**LSP (label switched path).** The one-directional path of a labelled packet from ingress to egress.

**LSR (label switching router).** A router that forwards packets by label.

**Mapping server.** In SR, a router that advertises prefix SIDs on behalf of routers that cannot.

**Microloop.** A transient loop while routers update their tables at different times.

**MP-BGP.** Multiprotocol BGP, carrying address families such as VPNv4, VPNv6, and EVPN.

**MSD (maximum SID depth).** How many labels or SIDs a router can push.

**Node SID.** A prefix SID on a router's loopback, meaning "the shortest path to this router".

**P (provider) router.** A core router that only switches labels.

**PCE, PCEP.** Path computation element, a controller that computes TE paths, and the protocol it uses to talk to head ends.

**PE (provider edge).** A provider router facing customers; it holds VRFs and pushes service labels.

**PHP (penultimate hop popping).** The router before the egress pops the last label.

**PLR (point of local repair).** The router next to a failure, which switches traffic to a backup.

**Prefix SID.** A global segment attached to a prefix.

**Pseudowire (PW).** An emulated point-to-point circuit over an MPLS network.

**PQ node.** In remote LFA, a router that is both reachable from the PLR and able to reach the destination without the failed link.

**Push, swap, pop.** The three label operations: add, replace, and remove the top label.

**RD (route distinguisher).** A 64-bit value making VPN prefixes unique in BGP. It carries no policy.

**Remote LFA.** Fast reroute by tunnelling to a PQ node (RFC 7490).

**Route reflector (RR).** A BGP router that re-advertises iBGP routes, removing the need for a full mesh.

**RSVP-TE.** A signalling protocol that builds LSPs along explicit paths with bandwidth reservation (RFC 3209).

**RT (route target).** An extended community that controls which VRFs import a VPN route.

**Segment, SID.** An instruction in a segment routing path, and its identifier: a label in SR-MPLS, an IPv6 address in SRv6.

**Segment Routing (SR).** Source routing in which the ingress encodes the path as a list of segments (RFC 8402).

**Site of Origin (SoO).** An extended community preventing routes from being sent back to the site they came from.

**Split horizon.** In VPLS, never forwarding a frame from one pseudowire to another.

**SR policy.** A head end's set of candidate paths to an endpoint for a color (RFC 9256).

**SRGB.** Segment Routing Global Block: the label range reserved for prefix SIDs.

**SRH.** Segment Routing Header, the IPv6 extension header that lists SRv6 segments (RFC 8754).

**SRv6.** Segment Routing over IPv6 (RFC 8986).

**TC.** Traffic class: the 3 QoS bits of a label stack entry, formerly called EXP.

**TI-LFA.** Topology-independent LFA: fast reroute along the post-convergence path, encoded as a segment list.

**Transport label.** The outer label that carries a packet to the egress PE.

**TTL.** Time to live, decremented at each hop to stop loops.

**uSID.** Micro-SID: a short SRv6 SID packed with others into one IPv6 address (RFC 9800).

**VPLS.** Virtual private LAN service: a multipoint Ethernet VPN over a mesh of pseudowires.

**VPN label.** The inner label that tells the egress PE which VRF or circuit a packet belongs to.

**VPNv4, VPNv6.** BGP address families carrying RD-prefixed VPN routes.

**VPWS.** Virtual private wire service: a point-to-point Ethernet VPN built from a pseudowire.

**VRF.** Virtual routing and forwarding instance: a separate routing table with its interfaces.
