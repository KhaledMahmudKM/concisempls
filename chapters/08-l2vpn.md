# Chapter 8: Layer 2 VPNs

Not every customer wants the provider to route their traffic. Some want a plain Ethernet connection between two sites, or a single virtual LAN that spans many sites, so their own routers (or even their own switches) see each other as directly connected. MPLS provides these **Layer 2 VPNs** with the same tools as before: a transport label to cross the core, and a second label that identifies the customer's circuit.

> **No lab.** Pseudowires, VPLS and EVPN over MPLS have no data plane in the Linux kernel, so they cannot be built with FRR containers. FRR's LDP can signal pseudowires only on OpenBSD. This chapter therefore uses diagrams only. The closest thing you can run is EVPN over VXLAN, which uses the same BGP control plane; the Concise Book on Data Center Fabric covers it with FRR labs.

## Pseudowires

A **pseudowire (PW)** emulates a point-to-point link across a packet network.

<figure class="diagram">
<svg viewBox="0 0 720 250" style="width:100%;height:auto;max-width:720px" role="img" aria-label="A pseudowire: CE1 connects to PE1 over an attachment circuit; PE1 and PE2 are joined by a pseudowire carried inside an MPLS tunnel across the core; PE2 delivers frames to CE2 over its attachment circuit. The two CEs behave as if one Ethernet cable connected them." xmlns="http://www.w3.org/2000/svg">
<style>
.m8a{ font-family:var(--sans),sans-serif; }
.m8a .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.m8a .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.m8a .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.m8a .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.m8a .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.m8a .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.m8a .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.m8a .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.m8a .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.m8a .h{ fill:var(--text); font-size:15px; font-weight:700; }
.m8a .b{ fill:var(--text); font-size:13px; font-weight:700; }
.m8a .s{ fill:var(--text-dim); font-size:13px; }
.m8a .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.m8a .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.m8a .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.m8a .c{ fill:var(--text-faint); font-size:12.5px; }
.m8a .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.m8a .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.m8a .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.m8a .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m8a-a); }
.m8a .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m8a-a); marker-start:url(#m8a-s); }
.m8a .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#m8a-g); }
.m8a .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#m8a-a); }
.m8a .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.m8a .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.m8a .bad{ stroke:var(--text); stroke-width:3; fill:none; }
</style>
<defs><marker id="m8a-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="m8a-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="m8a-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker></defs>
<g class="m8a">
<circle cx="60" cy="110" r="24" class="acc"/>
<text x="60" y="114.5" text-anchor="middle" class="b">CE1</text>
<circle cx="660" cy="110" r="24" class="acc"/>
<text x="660" y="114.5" text-anchor="middle" class="b">CE2</text>
<circle cx="180" cy="110" r="24" class="dark"/>
<text x="180" y="114.5" text-anchor="middle" class="on" style="fill:var(--bg)">PE1</text>
<circle cx="540" cy="110" r="24" class="dark"/>
<text x="540" y="114.5" text-anchor="middle" class="on" style="fill:var(--bg)">PE2</text>
<line x1="84" y1="110" x2="156" y2="110" class="lk"/>
<line x1="564" y1="110" x2="636" y2="110" class="lk"/>
<rect x="204" y="86" width="312" height="48" rx="7" class="tint"/>
<line x1="204" y1="110" x2="516" y2="110" class="dash"/>
<text x="360" y="76" text-anchor="middle" class="c">MPLS tunnel (LDP or SR LSP)</text>
<text x="360" y="100" text-anchor="middle" class="a">pseudowire</text>
<text x="120" y="140" text-anchor="middle" class="c">AC</text>
<text x="600" y="140" text-anchor="middle" class="c">AC</text>
<text x="360" y="180" text-anchor="middle" class="c">AC = attachment circuit: a port or VLAN facing the customer</text>
<text x="360" y="206" text-anchor="middle" class="s">to the CEs it looks like one long Ethernet cable</text>
</g>
</svg>
<figcaption>A pseudowire emulates a point-to-point link. VPWS (E-Line) services are built from them.</figcaption>
</figure>

The customer connects to each PE through an **attachment circuit (AC)**: a physical port, or a VLAN on a port. The PE takes every frame arriving on the AC and sends it, unchanged, through the pseudowire to the far PE, which delivers it on its own AC. The two CEs behave as if one long Ethernet cable joined them. Services built this way are called **VPWS** (virtual private wire service), or **E-Line** in Metro Ethernet Forum terms.

<figure class="diagram">
<svg viewBox="0 0 720 170" style="width:100%;height:auto;max-width:720px" role="img" aria-label="Pseudowire encapsulation: the transport label to reach the remote PE, the pseudowire label that identifies the circuit, an optional 4-byte control word, and the customer&#x27;s entire Ethernet frame." xmlns="http://www.w3.org/2000/svg">
<style>
.m8b{ font-family:var(--sans),sans-serif; }
.m8b .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.m8b .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.m8b .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.m8b .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.m8b .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.m8b .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.m8b .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.m8b .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.m8b .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.m8b .h{ fill:var(--text); font-size:15px; font-weight:700; }
.m8b .b{ fill:var(--text); font-size:13px; font-weight:700; }
.m8b .s{ fill:var(--text-dim); font-size:13px; }
.m8b .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.m8b .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.m8b .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.m8b .c{ fill:var(--text-faint); font-size:12.5px; }
.m8b .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.m8b .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.m8b .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.m8b .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m8b-a); }
.m8b .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m8b-a); marker-start:url(#m8b-s); }
.m8b .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#m8b-g); }
.m8b .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#m8b-a); }
.m8b .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.m8b .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.m8b .bad{ stroke:var(--text); stroke-width:3; fill:none; }
</style>
<defs><marker id="m8b-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="m8b-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="m8b-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker></defs>
<g class="m8b">
<rect x="30" y="50" width="140" height="34" rx="3" class="acc"/>
<text x="100.0" y="71.5" text-anchor="middle" class="m">transport label</text>
<rect x="170" y="50" width="110" height="34" rx="3" class="dark"/>
<text x="225.0" y="71.5" text-anchor="middle" class="on" style="fill:var(--bg)">PW label</text>
<rect x="280" y="50" width="120" height="34" rx="3" class="pnl"/>
<text x="340.0" y="71.5" text-anchor="middle" class="m">control word</text>
<rect x="400" y="50" width="290" height="34" rx="3" class="box"/>
<text x="545.0" y="71.5" text-anchor="middle" class="m">customer Ethernet frame</text>
<text x="100" y="112" text-anchor="middle" class="c">to the remote PE</text>
<text x="195" y="112" text-anchor="middle" class="c">which circuit</text>
<text x="310" y="112" text-anchor="middle" class="c">optional</text>
<text x="515" y="112" text-anchor="middle" class="c">MACs, VLAN tag, payload</text>
<text x="360" y="150" text-anchor="middle" class="c">the PW label is signalled by targeted LDP (RFC 8077) or BGP</text>
</g>
</svg>
<figcaption>Two labels again: one to cross the core, one to identify the customer circuit at the far end.</figcaption>
</figure>

The encapsulation should look familiar:

- The **transport label** takes the packet to the remote PE over an LDP or Segment Routing LSP.
- The **PW label** identifies the circuit at the remote PE. It is allocated by that PE and signalled with **targeted LDP** (a session between non-adjacent PEs, RFC 8077) or with BGP.
- An optional 4-byte **control word** carries a sequence number and, more importantly, stops core routers from mistaking the Ethernet payload for an IP packet when they look deeper for load balancing.
- The customer's **entire Ethernet frame** follows: MAC addresses, VLAN tags, payload.

A PE does not learn MAC addresses on a pseudowire; whatever enters one end leaves the other.

## VPLS: a virtual switch

Many customers want more than two sites on one LAN. **VPLS** (virtual private LAN service, or **E-LAN**) connects any number of sites into a single Ethernet broadcast domain.

<figure class="diagram">
<svg viewBox="0 0 720 290" style="width:100%;height:auto;max-width:720px" role="img" aria-label="VPLS: three PEs joined by a full mesh of pseudowires form one virtual switch for a customer. Each PE learns MAC addresses from frames it receives and floods unknown destinations. Split horizon stops a frame received on one pseudowire from being sent out another pseudowire." xmlns="http://www.w3.org/2000/svg">
<style>
.m8c{ font-family:var(--sans),sans-serif; }
.m8c .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.m8c .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.m8c .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.m8c .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.m8c .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.m8c .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.m8c .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.m8c .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.m8c .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.m8c .h{ fill:var(--text); font-size:15px; font-weight:700; }
.m8c .b{ fill:var(--text); font-size:13px; font-weight:700; }
.m8c .s{ fill:var(--text-dim); font-size:13px; }
.m8c .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.m8c .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.m8c .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.m8c .c{ fill:var(--text-faint); font-size:12.5px; }
.m8c .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.m8c .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.m8c .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.m8c .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m8c-a); }
.m8c .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m8c-a); marker-start:url(#m8c-s); }
.m8c .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#m8c-g); }
.m8c .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#m8c-a); }
.m8c .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.m8c .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.m8c .bad{ stroke:var(--text); stroke-width:3; fill:none; }
</style>
<defs><marker id="m8c-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="m8c-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="m8c-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker></defs>
<g class="m8c">
<line x1="213.0" y1="80.0" x2="507.0" y2="80.0" class="dash"/>
<line x1="205.4" y1="101.1" x2="334.6" y2="208.9" class="dash"/>
<line x1="514.6" y1="101.1" x2="385.4" y2="208.9" class="dash"/>
<circle cx="180" cy="80" r="32" class="dark"/>
<text x="180" y="85" text-anchor="middle" class="on" style="fill:var(--bg)">PE1</text>
<circle cx="540" cy="80" r="32" class="dark"/>
<text x="540" y="85" text-anchor="middle" class="on" style="fill:var(--bg)">PE2</text>
<circle cx="360" cy="230" r="32" class="dark"/>
<text x="360" y="235" text-anchor="middle" class="on" style="fill:var(--bg)">PE3</text>
<circle cx="50" cy="80" r="24" class="acc"/>
<text x="50" y="84.5" text-anchor="middle" class="b">CE1</text>
<line x1="74" y1="80" x2="146" y2="80" class="lk"/>
<circle cx="670" cy="80" r="24" class="acc"/>
<text x="670" y="84.5" text-anchor="middle" class="b">CE2</text>
<line x1="646" y1="80" x2="574" y2="80" class="lk"/>
<circle cx="560" cy="230" r="24" class="acc"/>
<text x="560" y="234.5" text-anchor="middle" class="b">CE3</text>
<line x1="536" y1="230" x2="394" y2="230" class="lk"/>
<text x="360" y="66" text-anchor="middle" class="a">full mesh of pseudowires</text>
<rect x="20" y="140" width="230" height="64" rx="7" class="pnl"/>
<text x="135.0" y="167.5" text-anchor="middle" class="b">each PE: a MAC table</text>
<text x="135.0" y="185.5" text-anchor="middle" class="c">learn, flood, age out</text>
<text x="150" y="262" text-anchor="middle" class="c">split horizon: never PW to PW</text>
</g>
</svg>
<figcaption>VPLS turns the provider network into one big Ethernet switch per customer (E-LAN).</figcaption>
</figure>

Each PE runs a virtual switch per customer, called a VSI (virtual switching instance), connected to every other PE in that VPN by a pseudowire. Like any switch, the VSI:

- **learns** source MAC addresses from frames, recording which AC or pseudowire they came from;
- **forwards** frames to known destinations on the right AC or pseudowire;
- **floods** broadcast, multicast, and unknown unicast frames to every AC and pseudowire.

Because the pseudowires form a full mesh, a frame received from one pseudowire never needs to go out another. That rule, **split horizon**, replaces the spanning tree protocol inside the provider network and prevents loops.

VPLS comes in two signalling flavours: **LDP-signalled** (RFC 4762), using targeted LDP between every pair of PEs, and **BGP-signalled** (RFC 4761), using BGP for auto-discovery and label distribution. Large deployments use **hierarchical VPLS (H-VPLS)**, where small access PEs connect by a single pseudowire to a core PE instead of joining the full mesh.

VPLS has well-known weaknesses. The full mesh of pseudowires grows with the square of the PEs; MAC learning in the data plane means every PE floods unknown traffic; and a CE connected to two PEs can only use one of them at a time, or loops form.

## EVPN

**EVPN** (RFC 7432) fixes those weaknesses by moving MAC learning into the **control plane**. PEs advertise the MAC addresses they learn from their local customers in BGP, using the same RD and RT machinery as L3VPN, and remote PEs install them directly instead of discovering them by flooding.

<figure class="diagram">
<svg viewBox="0 0 720 280" style="width:100%;height:auto;max-width:720px" role="img" aria-label="EVPN: PEs advertise customer MAC addresses in BGP instead of learning them in the data plane. A CE can be multihomed to two PEs in all-active mode, sharing one Ethernet segment identifier, and remote PEs load-balance to both." xmlns="http://www.w3.org/2000/svg">
<style>
.m8d{ font-family:var(--sans),sans-serif; }
.m8d .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.m8d .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.m8d .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.m8d .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.m8d .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.m8d .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.m8d .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.m8d .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.m8d .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.m8d .h{ fill:var(--text); font-size:15px; font-weight:700; }
.m8d .b{ fill:var(--text); font-size:13px; font-weight:700; }
.m8d .s{ fill:var(--text-dim); font-size:13px; }
.m8d .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.m8d .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.m8d .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.m8d .c{ fill:var(--text-faint); font-size:12.5px; }
.m8d .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.m8d .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.m8d .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.m8d .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m8d-a); }
.m8d .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m8d-a); marker-start:url(#m8d-s); }
.m8d .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#m8d-g); }
.m8d .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#m8d-a); }
.m8d .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.m8d .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.m8d .bad{ stroke:var(--text); stroke-width:3; fill:none; }
</style>
<defs><marker id="m8d-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="m8d-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="m8d-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker></defs>
<g class="m8d">
<circle cx="60" cy="140" r="24" class="acc"/>
<text x="60" y="144.5" text-anchor="middle" class="b">CE1</text>
<circle cx="220" cy="80" r="24" class="dark"/>
<text x="220" y="84.5" text-anchor="middle" class="on" style="fill:var(--bg)">PE1</text>
<circle cx="220" cy="200" r="24" class="dark"/>
<text x="220" y="204.5" text-anchor="middle" class="on" style="fill:var(--bg)">PE2</text>
<circle cx="560" cy="140" r="24" class="dark"/>
<text x="560" y="144.5" text-anchor="middle" class="on" style="fill:var(--bg)">PE3</text>
<circle cx="670" cy="140" r="24" class="acc"/>
<text x="670" y="144.5" text-anchor="middle" class="b">CE2</text>
<line x1="80" y1="128" x2="198" y2="88" class="lk"/>
<line x1="80" y1="152" x2="198" y2="192" class="lk"/>
<line x1="584" y1="140" x2="646" y2="140" class="lk"/>
<rect x="20" y="60" width="140" height="160" rx="7" class="zone"/>
<text x="90" y="236" text-anchor="middle" class="c">ES 01:..:01, all-active</text>
<path d="M244 80 C 380 60, 470 90, 538 128" class="ad"/>
<path d="M244 200 C 380 220, 470 190, 538 152" class="ad"/>
<rect x="300" y="116" width="200" height="48" rx="7" class="pnl"/>
<text x="400.0" y="144.5" text-anchor="middle" class="b">BGP: MAC, IP, ES routes</text>
<text x="390" y="52" text-anchor="middle" class="c">type 2: MAC 00:aa:.. via PE1</text>
<text x="390" y="244" text-anchor="middle" class="c">type 2: same MAC via PE2</text>
</g>
</svg>
<figcaption>EVPN moves MAC learning into BGP. Multihoming, fast convergence and ARP suppression come with it.</figcaption>
</figure>

EVPN defines several BGP route types. The main ones are:

| Type | Name | Purpose |
|---|---|---|
| 1 | Ethernet auto-discovery | Multihoming: which PEs serve an Ethernet segment; fast mass withdrawal |
| 2 | MAC/IP advertisement | A customer MAC address, optionally with its IP address |
| 3 | Inclusive multicast | Where to send broadcast and unknown traffic for each VPN |
| 4 | Ethernet segment | Elects which PE forwards broadcast traffic to a multihomed CE |
| 5 | IP prefix | Routes for layer 3 services in the same framework |

The benefits follow directly:

- **All-active multihoming.** A CE can connect to two or more PEs at once. The links share an **Ethernet segment identifier (ESI)**, and remote PEs load-balance across all of them.
- **Fast convergence.** When a link to a multihomed site fails, one type 1 withdrawal moves all its MAC addresses at once.
- **Less flooding.** Type 2 routes can carry IP addresses, so PEs can answer ARP requests locally (ARP suppression).
- **One framework** for layer 2 and layer 3 services, and for point-to-point circuits too: **EVPN-VPWS** (RFC 8214) replaces classic pseudowires.

EVPN is independent of the data plane. Over MPLS, it uses a transport label and a service label, as everywhere else in this book. In data centres, the same control plane usually runs over **VXLAN**, where a 24-bit VNI plays the role of the service label. New layer 2 VPN deployments, in both service providers and data centres, almost always choose EVPN.

## Choosing a layer 2 service

| Need | Classic | Modern |
|---|---|---|
| Point-to-point Ethernet | VPWS pseudowire (LDP) | EVPN-VPWS |
| Multipoint Ethernet | VPLS (LDP or BGP) | EVPN |
| Active-active multihoming | Not possible | EVPN all-active |
| Mixed layer 2 and layer 3 | Separate services | EVPN with type 5 routes |

## Summary

- A **pseudowire** emulates a point-to-point link: **transport label**, **PW label**, optional **control word**, then the customer's **frame**.
- **VPWS** (E-Line) is built from one pseudowire; **VPLS** (E-LAN) from a full mesh of them, with data-plane MAC learning and **split horizon**.
- **EVPN** advertises MAC addresses in **BGP**, enabling **all-active multihoming**, fast convergence, and less flooding.
- EVPN runs over MPLS or VXLAN; new deployments favour it over VPLS and classic pseudowires.
- None of these has a Linux data plane over MPLS; practise the EVPN control plane with **EVPN-VXLAN** in FRR instead.
