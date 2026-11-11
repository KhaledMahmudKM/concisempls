# Chapter 5: The BGP-Free Core

An Internet service provider carries the full Internet routing table, more than a million prefixes, and customer routes on top. In a pure IP network, every router on the path needs those routes, so every core router must run BGP and hold the full table. With MPLS, only the edge routers need them. This chapter shows how, and introduces two related techniques: 6PE for IPv6, and BGP labeled unicast for very large networks.

## The idea

Imagine pe1 receives a packet for `192.168.2.1`, a destination behind pe2. pe1 knows the route through BGP: `192.168.2.0/24`, next hop `10.0.0.2`, pe2's loopback. Without MPLS, pe1 would send the IP packet toward p1, and p1 would need its own route to `192.168.2.0/24`.

With MPLS, pe1 instead puts the packet on the LSP toward `10.0.0.2`. The core switches on that label alone and delivers the packet to pe2, which then does the IP lookup.

<figure class="diagram">
<svg viewBox="0 0 720 290" style="width:100%;height:auto;max-width:720px" role="img" aria-label="A BGP-free core: pe1 and pe2 run iBGP between their loopbacks and know the customer route 192.168.2.0/24; p1 and p2 run only IS-IS and LDP. A packet from ce1 to 192.168.2.1 gets the LDP label for 10.0.0.2 at pe1 and crosses the core on that label." xmlns="http://www.w3.org/2000/svg">
<style>
.m5a{ font-family:var(--sans),sans-serif; }
.m5a .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.m5a .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.m5a .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.m5a .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.m5a .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.m5a .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.m5a .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.m5a .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.m5a .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.m5a .h{ fill:var(--text); font-size:15px; font-weight:700; }
.m5a .b{ fill:var(--text); font-size:13px; font-weight:700; }
.m5a .s{ fill:var(--text-dim); font-size:13px; }
.m5a .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.m5a .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.m5a .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.m5a .c{ fill:var(--text-faint); font-size:12.5px; }
.m5a .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.m5a .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.m5a .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.m5a .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m5a-a); }
.m5a .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m5a-a); marker-start:url(#m5a-s); }
.m5a .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#m5a-g); }
.m5a .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#m5a-a); }
.m5a .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.m5a .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.m5a .bad{ stroke:var(--text); stroke-width:3; fill:none; }
</style>
<defs><marker id="m5a-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="m5a-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="m5a-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker></defs>
<g class="m5a">
<circle cx="60" cy="130" r="24" class="pnl"/>
<text x="60" y="134.5" text-anchor="middle" class="b">ce1</text>
<circle cx="180" cy="130" r="24" class="dark"/>
<text x="180" y="134.5" text-anchor="middle" class="on" style="fill:var(--bg)">pe1</text>
<circle cx="300" cy="130" r="24" class="pnl"/>
<text x="300" y="134.5" text-anchor="middle" class="b">p1</text>
<circle cx="420" cy="130" r="24" class="pnl"/>
<text x="420" y="134.5" text-anchor="middle" class="b">p2</text>
<circle cx="540" cy="130" r="24" class="dark"/>
<text x="540" y="134.5" text-anchor="middle" class="on" style="fill:var(--bg)">pe2</text>
<circle cx="660" cy="130" r="24" class="pnl"/>
<text x="660" y="134.5" text-anchor="middle" class="b">ce2</text>
<line x1="84" y1="130" x2="156" y2="130" class="lk"/>
<line x1="204" y1="130" x2="276" y2="130" class="lk"/>
<line x1="324" y1="130" x2="396" y2="130" class="lk"/>
<line x1="444" y1="130" x2="516" y2="130" class="lk"/>
<line x1="564" y1="130" x2="636" y2="130" class="lk"/>
<path d="M180 100 C 240 30, 420 30, 480 100" class="ad"/>
<text x="330" y="34" text-anchor="middle" class="a">iBGP 10.0.0.1 to 10.0.0.2: 192.168.2.0/24 via 10.0.0.2</text>
<rect x="240" y="168" width="180" height="46" rx="7" class="pnl"/>
<text x="330" y="188" text-anchor="middle" class="b">p1, p2: IS-IS + LDP only</text>
<text x="330" y="204" text-anchor="middle" class="c">no route to 192.168.2.0/24</text>
<rect x="196" y="236" width="140" height="28" rx="3" class="acc"/>
<text x="266.0" y="254.5" text-anchor="middle" class="m">label to 10.0.0.2</text>
<rect x="336" y="236" width="140" height="28" rx="3" class="box"/>
<text x="406.0" y="254.5" text-anchor="middle" class="m">IP to 192.168.2.1</text>
<text x="330" y="284" text-anchor="middle" class="c">the core switches on the outer label and never reads the IP header</text>
</g>
</svg>
<figcaption>Only the edges know customer and Internet routes. The core needs routes and labels for the PE loopbacks, nothing else.</figcaption>
</figure>

The core routers need routes and labels only for the PE loopbacks. They run the IGP and LDP, but **no BGP**. In a large network, that is the difference between core routers holding a few thousand routes and holding millions.

## Recursive resolution

The only new mechanism is in the PE, and it is a familiar one: **recursive next-hop resolution**.

<figure class="diagram">
<svg viewBox="0 0 720 250" style="width:100%;height:auto;max-width:720px" role="img" aria-label="Route resolution on pe1: the BGP route 192.168.2.0/24 has next hop 10.0.0.2; IS-IS reaches 10.0.0.2 through p1 at 10.10.1.2 on eth1; LDP says p1&#x27;s label for 10.0.0.2 is 22; so zebra installs 192.168.2.0/24 via 10.10.1.2 dev eth1 with label 22 pushed." xmlns="http://www.w3.org/2000/svg">
<style>
.m5b{ font-family:var(--sans),sans-serif; }
.m5b .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.m5b .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.m5b .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.m5b .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.m5b .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.m5b .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.m5b .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.m5b .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.m5b .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.m5b .h{ fill:var(--text); font-size:15px; font-weight:700; }
.m5b .b{ fill:var(--text); font-size:13px; font-weight:700; }
.m5b .s{ fill:var(--text-dim); font-size:13px; }
.m5b .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.m5b .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.m5b .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.m5b .c{ fill:var(--text-faint); font-size:12.5px; }
.m5b .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.m5b .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.m5b .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.m5b .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m5b-a); }
.m5b .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m5b-a); marker-start:url(#m5b-s); }
.m5b .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#m5b-g); }
.m5b .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#m5b-a); }
.m5b .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.m5b .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.m5b .bad{ stroke:var(--text); stroke-width:3; fill:none; }
</style>
<defs><marker id="m5b-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="m5b-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="m5b-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker></defs>
<g class="m5b">
<rect x="20" y="30" width="160" height="76" rx="7" class="pnl"/>
<text x="100.0" y="54.5" text-anchor="middle" class="b">BGP</text>
<text x="100.0" y="72.5" text-anchor="middle" class="m">192.168.2.0/24</text>
<text x="100.0" y="90.5" text-anchor="middle" class="c">next hop 10.0.0.2</text>
<rect x="200" y="30" width="160" height="76" rx="7" class="pnl"/>
<text x="280.0" y="54.5" text-anchor="middle" class="b">IS-IS</text>
<text x="280.0" y="72.5" text-anchor="middle" class="m">10.0.0.2/32</text>
<text x="280.0" y="90.5" text-anchor="middle" class="c">via 10.10.1.2, eth1</text>
<line x1="182" y1="68" x2="198" y2="68" class="ar"/>
<rect x="380" y="30" width="160" height="76" rx="7" class="pnl"/>
<text x="460.0" y="54.5" text-anchor="middle" class="b">LDP</text>
<text x="460.0" y="72.5" text-anchor="middle" class="m">p1's label for it</text>
<text x="460.0" y="90.5" text-anchor="middle" class="c">22</text>
<line x1="362" y1="68" x2="378" y2="68" class="ar"/>
<rect x="560" y="30" width="150" height="76" rx="7" class="dark"/>
<text x="635.0" y="54.5" text-anchor="middle" class="on" style="fill:var(--bg)">installed</text>
<text x="635.0" y="72.5" text-anchor="middle" class="on" style="fill:var(--bg)">push 22</text>
<text x="635.0" y="90.5" text-anchor="middle" class="on" style="fill:var(--bg)">out eth1</text>
<line x1="542" y1="68" x2="558" y2="68" class="ar"/>
<rect x="20" y="140" width="690" height="50" rx="7" class="acc"/>
<text x="365.0" y="169.5" text-anchor="middle" class="m">ip route on pe1:  192.168.2.0/24 encap mpls 22 via 10.10.1.2 dev eth1</text>
<text x="365" y="218" text-anchor="middle" class="c">BGP never deals with labels here: the label comes from resolving the next hop.</text>
</g>
</svg>
<figcaption>Recursive resolution is the whole trick. The BGP next hop is a PE loopback, and the loopback has an LSP.</figcaption>
</figure>

When BGP installs `192.168.2.0/24` with next hop `10.0.0.2`, the routing table resolves that next hop through the IGP route to `10.0.0.2/32`. That route has an outgoing interface, a next-hop address, and, because LDP is running, a label. The installed entry inherits all three. BGP itself never deals with the label. It is a side effect of the BGP next hop being a PE loopback with an LSP.

Two details make this work:

- **The BGP next hop must be the PE's loopback.** For routes a PE learned from a customer or from an eBGP peer, the next hop is initially the neighbour's address, which the core does not know. Configure **`next-hop-self`** on the iBGP sessions so every route a PE advertises points at its own loopback.
- **The loopback must have an LSP from every PE to every PE.** If any hop on the way is missing a label, the packet is sent unlabelled, and an unlabelled packet in a BGP-free core is lost.

<figure class="diagram">
<svg viewBox="0 0 720 210" style="width:100%;height:auto;max-width:720px" role="img" aria-label="What happens without an LSP: if p1 has no LDP label for 10.0.0.2, pe1 sends the customer packet unlabelled; p1 looks up 192.168.2.1, finds no route because it runs no BGP, and drops the packet." xmlns="http://www.w3.org/2000/svg">
<style>
.m5c{ font-family:var(--sans),sans-serif; }
.m5c .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.m5c .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.m5c .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.m5c .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.m5c .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.m5c .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.m5c .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.m5c .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.m5c .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.m5c .h{ fill:var(--text); font-size:15px; font-weight:700; }
.m5c .b{ fill:var(--text); font-size:13px; font-weight:700; }
.m5c .s{ fill:var(--text-dim); font-size:13px; }
.m5c .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.m5c .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.m5c .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.m5c .c{ fill:var(--text-faint); font-size:12.5px; }
.m5c .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.m5c .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.m5c .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.m5c .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m5c-a); }
.m5c .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m5c-a); marker-start:url(#m5c-s); }
.m5c .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#m5c-g); }
.m5c .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#m5c-a); }
.m5c .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.m5c .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.m5c .bad{ stroke:var(--text); stroke-width:3; fill:none; }
</style>
<defs><marker id="m5c-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="m5c-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="m5c-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker></defs>
<g class="m5c">
<circle cx="90" cy="70" r="24" class="dark"/>
<text x="90" y="74.5" text-anchor="middle" class="on" style="fill:var(--bg)">pe1</text>
<circle cx="270" cy="70" r="24" class="pnl"/>
<text x="270" y="74.5" text-anchor="middle" class="b">p1</text>
<circle cx="450" cy="70" r="24" class="pnl"/>
<text x="450" y="74.5" text-anchor="middle" class="b">p2</text>
<circle cx="630" cy="70" r="24" class="dark"/>
<text x="630" y="74.5" text-anchor="middle" class="on" style="fill:var(--bg)">pe2</text>
<line x1="114" y1="70" x2="246" y2="70" class="lk"/>
<line x1="294" y1="70" x2="426" y2="70" class="lk"/>
<line x1="474" y1="70" x2="606" y2="70" class="lk"/>
<rect x="118" y="112" width="140" height="28" rx="3" class="box"/>
<text x="188.0" y="130.5" text-anchor="middle" class="m">IP to 192.168.2.1</text>
<rect x="330" y="108" width="380" height="48" rx="7" class="pnl"/>
<text x="520.0" y="136.5" text-anchor="middle" class="b">p1: no route to 192.168.2.1, packet dropped</text>
<text x="360" y="186" text-anchor="middle" class="c">A BGP-free core is only safe when every PE loopback has an LSP end to end.</text>
</g>
</svg>
<figcaption>Unlabelled customer traffic is lost in a BGP-free core. Keep the LSPs healthy, or keep BGP in the core.</figcaption>
</figure>

That second point is why LDP-IGP synchronisation (Chapter 4) matters so much in BGP-free cores, and why operators monitor LSPs between all PE pairs.

## IPv6 over an IPv4 core: 6PE

The same trick carries IPv6 across a core that only runs IPv4 and LDP. **6PE** (RFC 4798) advertises IPv6 routes between PEs over iBGP sessions between their IPv4 loopbacks, and attaches a **label** to each route.

<figure class="diagram">
<svg viewBox="0 0 720 250" style="width:100%;height:auto;max-width:720px" role="img" aria-label="6PE: IPv6 sites connect over an IPv4-only MPLS core. pe1 learns the IPv6 route from pe2 over iBGP with a 6PE label, and sends IPv6 packets with two labels: the LDP label to pe2&#x27;s IPv4 loopback on top, and the 6PE label underneath." xmlns="http://www.w3.org/2000/svg">
<style>
.m5d{ font-family:var(--sans),sans-serif; }
.m5d .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.m5d .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.m5d .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.m5d .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.m5d .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.m5d .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.m5d .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.m5d .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.m5d .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.m5d .h{ fill:var(--text); font-size:15px; font-weight:700; }
.m5d .b{ fill:var(--text); font-size:13px; font-weight:700; }
.m5d .s{ fill:var(--text-dim); font-size:13px; }
.m5d .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.m5d .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.m5d .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.m5d .c{ fill:var(--text-faint); font-size:12.5px; }
.m5d .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.m5d .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.m5d .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.m5d .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m5d-a); }
.m5d .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m5d-a); marker-start:url(#m5d-s); }
.m5d .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#m5d-g); }
.m5d .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#m5d-a); }
.m5d .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.m5d .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.m5d .bad{ stroke:var(--text); stroke-width:3; fill:none; }
</style>
<defs><marker id="m5d-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="m5d-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="m5d-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker></defs>
<g class="m5d">
<rect x="170" y="40" width="380" height="120" rx="7" class="zone"/>
<text x="360" y="62" text-anchor="middle" class="c">IPv4-only MPLS core (IS-IS, LDP)</text>
<circle cx="70" cy="110" r="24" class="pnl"/>
<text x="70" y="114.5" text-anchor="middle" class="b">v6 site</text>
<circle cx="215" cy="110" r="24" class="dark"/>
<text x="215" y="114.5" text-anchor="middle" class="on" style="fill:var(--bg)">pe1</text>
<circle cx="360" cy="110" r="24" class="pnl"/>
<text x="360" y="114.5" text-anchor="middle" class="b">p</text>
<circle cx="505" cy="110" r="24" class="dark"/>
<text x="505" y="114.5" text-anchor="middle" class="on" style="fill:var(--bg)">pe2</text>
<circle cx="650" cy="110" r="24" class="pnl"/>
<text x="650" y="114.5" text-anchor="middle" class="b">v6 site</text>
<line x1="94" y1="110" x2="191" y2="110" class="lk"/>
<line x1="239" y1="110" x2="336" y2="110" class="lk"/>
<line x1="384" y1="110" x2="481" y2="110" class="lk"/>
<line x1="529" y1="110" x2="626" y2="110" class="lk"/>
<rect x="160" y="190" width="120" height="28" rx="3" class="acc"/>
<text x="220.0" y="208.5" text-anchor="middle" class="m">LDP: to pe2</text>
<rect x="280" y="190" width="100" height="28" rx="3" class="acc"/>
<text x="330.0" y="208.5" text-anchor="middle" class="m">6PE label</text>
<rect x="380" y="190" width="150" height="28" rx="3" class="box"/>
<text x="455.0" y="208.5" text-anchor="middle" class="m">IPv6 packet</text>
<text x="220" y="238" text-anchor="middle" class="c">IPv4 LSP</text>
<text x="330" y="238" text-anchor="middle" class="c">says IPv6</text>
</g>
</svg>
<figcaption>6PE carries IPv6 without upgrading the core. The bottom label tells pe2 the payload is IPv6.</figcaption>
</figure>

The ingress PE pushes two labels: the LDP label toward the egress PE's IPv4 loopback, and underneath it the 6PE label. The second label exists for a subtle reason. PHP removes the top label before the egress, and without another label the egress would receive a bare IPv6 packet from a link that it may not even have IPv6 enabled on. The bottom label tells the egress unambiguously that the payload is IPv6. A close relative, **6VPE**, does the same for IPv6 VPN routes (Chapter 6).

## Larger networks: BGP labeled unicast

LDP needs every PE loopback in one IGP. Very large networks split into several IGP domains (regions, or metro and core), and loopbacks are not shared between them. **BGP labeled unicast** (BGP-LU, RFC 8277) solves this by advertising a PE loopback, with a label, in BGP across domain boundaries.

<figure class="diagram">
<svg viewBox="0 0 720 260" style="width:100%;height:auto;max-width:720px" role="img" aria-label="BGP labeled unicast across two IGP domains: each domain runs its own IGP and LDP; border routers advertise PE loopbacks with labels in BGP. pe1 sends a packet with three labels: the LDP label to the border router, the BGP-LU label for pe2, and the service label." xmlns="http://www.w3.org/2000/svg">
<style>
.m5e{ font-family:var(--sans),sans-serif; }
.m5e .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.m5e .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.m5e .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.m5e .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.m5e .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.m5e .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.m5e .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.m5e .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.m5e .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.m5e .h{ fill:var(--text); font-size:15px; font-weight:700; }
.m5e .b{ fill:var(--text); font-size:13px; font-weight:700; }
.m5e .s{ fill:var(--text-dim); font-size:13px; }
.m5e .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.m5e .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.m5e .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.m5e .c{ fill:var(--text-faint); font-size:12.5px; }
.m5e .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.m5e .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.m5e .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.m5e .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m5e-a); }
.m5e .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m5e-a); marker-start:url(#m5e-s); }
.m5e .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#m5e-g); }
.m5e .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#m5e-a); }
.m5e .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.m5e .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.m5e .bad{ stroke:var(--text); stroke-width:3; fill:none; }
</style>
<defs><marker id="m5e-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="m5e-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="m5e-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker></defs>
<g class="m5e">
<rect x="20" y="30" width="330" height="110" rx="7" class="zone"/>
<text x="185" y="52" text-anchor="middle" class="c">domain A: IGP + LDP</text>
<rect x="370" y="30" width="330" height="110" rx="7" class="zone"/>
<text x="535" y="52" text-anchor="middle" class="c">domain B: IGP + LDP</text>
<circle cx="80" cy="100" r="24" class="dark"/>
<text x="80" y="104.5" text-anchor="middle" class="on" style="fill:var(--bg)">pe1</text>
<circle cx="210" cy="100" r="24" class="pnl"/>
<text x="210" y="104.5" text-anchor="middle" class="b">p</text>
<circle cx="360" cy="100" r="24" class="acc"/>
<text x="360" y="104.5" text-anchor="middle" class="b">BR</text>
<circle cx="510" cy="100" r="24" class="pnl"/>
<text x="510" y="104.5" text-anchor="middle" class="b">p</text>
<circle cx="640" cy="100" r="24" class="dark"/>
<text x="640" y="104.5" text-anchor="middle" class="on" style="fill:var(--bg)">pe2</text>
<line x1="104" y1="100" x2="186" y2="100" class="lk"/>
<line x1="234" y1="100" x2="336" y2="100" class="lk"/>
<line x1="384" y1="100" x2="486" y2="100" class="lk"/>
<line x1="534" y1="100" x2="616" y2="100" class="lk"/>
<rect x="100" y="168" width="110" height="28" rx="3" class="acc"/>
<text x="155.0" y="186.5" text-anchor="middle" class="m">LDP: to BR</text>
<rect x="210" y="168" width="120" height="28" rx="3" class="acc"/>
<text x="270.0" y="186.5" text-anchor="middle" class="m">BGP-LU: pe2</text>
<rect x="330" y="168" width="90" height="28" rx="3" class="acc"/>
<text x="375.0" y="186.5" text-anchor="middle" class="m">service</text>
<rect x="420" y="168" width="100" height="28" rx="3" class="box"/>
<text x="470.0" y="186.5" text-anchor="middle" class="m">payload</text>
<text x="360" y="226" text-anchor="middle" class="c">BGP labeled unicast (RFC 8277) stitches LSPs across domains that do not share an IGP</text>
</g>
</svg>
<figcaption>Labeled unicast extends the BGP-free idea across domains. It is the basis of large designs such as seamless MPLS.</figcaption>
</figure>

The ingress PE now pushes a three-level stack: an LDP label to reach the border router in its own domain, the BGP-LU label that the border router allocated for the remote PE, and the service label (for a VPN, for example). Each domain only needs its own IGP and LDP. This hierarchical approach is called **seamless MPLS**, and the same idea connects separate providers in inter-AS VPN option C (Chapter 7).

## Hands on: a BGP-free core

The lab `labs/lab3-bgp-free-core` is the LDP core from Chapter 4 with two customer sites: ce1 (`192.168.1.1`) behind pe1 and ce2 (`192.168.2.1`) behind pe2. The PEs reach the customers with static routes and exchange them over iBGP between their loopbacks. The P routers run only IS-IS and LDP.

The BGP configuration on pe1:

```text
router bgp 65000
 bgp router-id 10.0.0.1
 no bgp default ipv4-unicast
 neighbor 10.0.0.2 remote-as 65000
 neighbor 10.0.0.2 update-source lo
 !
 address-family ipv4 unicast
  redistribute static
  neighbor 10.0.0.2 activate
  neighbor 10.0.0.2 next-hop-self
 exit-address-family
exit
```

Deploy and test end to end:

```bash
cd labs/lab3-bgp-free-core
sudo containerlab deploy -t lab3-bgp-free-core.clab.yml
docker exec clab-lab3-bgp-free-core-ce1 ping -c 3 -I 192.168.1.1 192.168.2.1
```

Confirm where the knowledge lives:

```bash
docker exec clab-lab3-bgp-free-core-pe1 vtysh -c "show ip route 192.168.2.0/24"
docker exec clab-lab3-bgp-free-core-p1  vtysh -c "show ip route 192.168.2.0/24"
```

On pe1 the route is a BGP route, resolved recursively through `10.0.0.2`, with a label. On p1 there is **no route at all**, yet p1 forwards the traffic, because the traffic arrives with a label. Capture on p1 to see it:

```bash
sudo ip netns exec clab-lab3-bgp-free-core-p1 tcpdump -nni eth1 mpls
```

### Break the LSP

Disable LDP on p1 to see why the LSP is essential. In p1's `vtysh`:

```text
p1# configure terminal
p1(config)# no mpls ldp
```

pe1 no longer has a label from p1 for `10.0.0.2`, so it sends the customer packets unlabelled. p1 has no route to `192.168.2.1` and drops them; the ping from ce1 fails even though IS-IS is perfectly healthy. Restore it by pasting the `mpls ldp` block back from `configs/p1/frr.conf`.

## Summary

- In a **BGP-free core**, only PEs run BGP; P routers carry routes and labels for **PE loopbacks only**.
- It works through **recursive resolution**: the BGP next hop is a PE loopback, and the loopback's IGP route carries an LDP label.
- Use **`next-hop-self`** so every BGP next hop is a PE loopback.
- Any missing label on the path means **unlabelled traffic is dropped**; keep LSPs healthy and synchronised with the IGP.
- **6PE** carries IPv6 over an IPv4 MPLS core with a second label; **BGP labeled unicast** extends LSPs across IGP domains.
