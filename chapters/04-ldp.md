# Chapter 4: LDP

Static labels taught us the mechanics. A real network needs labels to appear automatically for every destination, follow the IGP when the topology changes, and disappear when routers fail. The **Label Distribution Protocol (LDP)** does exactly that: it gives every IGP prefix (in practice, every router loopback) a label on every router, and builds LSPs that follow the IGP's shortest paths.

## What LDP does, and does not do

LDP does **not** compute paths. The IGP (IS-IS or OSPF) still decides where packets go. LDP only attaches labels to the routes the IGP already chose. The FEC is simply an IP prefix, and the LSP for that prefix follows the IGP's best path toward it, hop by hop. If the IGP uses equal-cost paths, so does LDP.

This division of labour is LDP's great strength and its main limitation. It is simple and automatic, but it can never send traffic anywhere the IGP would not.

## Neighbours and sessions

LDP runs in two stages.

<figure class="diagram">
<svg viewBox="0 0 720 330" style="width:100%;height:auto;max-width:720px" role="img" aria-label="LDP session setup between p1 and p2 as a message sequence: both send Hello messages over UDP port 646 to 224.0.0.2 on the link; the router with the higher transport address opens a TCP connection to port 646; they exchange Initialization and Keepalive messages; then each sends Address and Label Mapping messages." xmlns="http://www.w3.org/2000/svg">
<style>
.m4a{ font-family:var(--sans),sans-serif; }
.m4a .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.m4a .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.m4a .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.m4a .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.m4a .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.m4a .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.m4a .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.m4a .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.m4a .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.m4a .h{ fill:var(--text); font-size:15px; font-weight:700; }
.m4a .b{ fill:var(--text); font-size:13px; font-weight:700; }
.m4a .s{ fill:var(--text-dim); font-size:13px; }
.m4a .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.m4a .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.m4a .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.m4a .c{ fill:var(--text-faint); font-size:12.5px; }
.m4a .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.m4a .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.m4a .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.m4a .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m4a-a); }
.m4a .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m4a-a); marker-start:url(#m4a-s); }
.m4a .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#m4a-g); }
.m4a .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#m4a-a); }
.m4a .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.m4a .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.m4a .bad{ stroke:var(--text); stroke-width:3; fill:none; }
</style>
<defs><marker id="m4a-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="m4a-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="m4a-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker></defs>
<g class="m4a">
<rect x="60" y="20" width="170" height="44" rx="7" class="pnl"/>
<text x="145.0" y="37.5" text-anchor="middle" class="b">p1</text>
<text x="145.0" y="55.5" text-anchor="middle" class="m">transport 10.0.0.11</text>
<rect x="490" y="20" width="170" height="44" rx="7" class="pnl"/>
<text x="575.0" y="37.5" text-anchor="middle" class="b">p2</text>
<text x="575.0" y="55.5" text-anchor="middle" class="m">transport 10.0.0.12</text>
<line x1="145" y1="64" x2="145" y2="316" class="gd"/>
<line x1="575" y1="64" x2="575" y2="316" class="gd"/>
<line x1="147" y1="92" x2="573" y2="92" class="ad"/>
<text x="360" y="85" text-anchor="middle" class="s">Hello (UDP 646 to 224.0.0.2)</text>
<line x1="573" y1="130" x2="147" y2="130" class="ar"/>
<text x="360" y="123" text-anchor="middle" class="s">TCP SYN to port 646 (higher address opens)</text>
<line x1="147" y1="168" x2="573" y2="168" class="ar2"/>
<text x="360" y="161" text-anchor="middle" class="s">Initialization (session parameters)</text>
<line x1="147" y1="206" x2="573" y2="206" class="ar2"/>
<text x="360" y="199" text-anchor="middle" class="s">Keepalive</text>
<line x1="147" y1="244" x2="573" y2="244" class="ar2"/>
<text x="360" y="237" text-anchor="middle" class="s">Address (my interface addresses)</text>
<line x1="147" y1="282" x2="573" y2="282" class="ar2"/>
<text x="360" y="275" text-anchor="middle" class="s">Label Mapping (FEC, label)</text>
<text x="30" y="96" text-anchor="start" class="c">discovery</text>
<text x="30" y="210" text-anchor="start" class="c">session</text>
<text x="30" y="286" text-anchor="start" class="c">labels</text>
</g>
</svg>
<figcaption>Discovery over UDP, then one TCP session per neighbour. Everything after Initialization rides that session.</figcaption>
</figure>

1. **Discovery.** Every LDP-enabled interface sends **Hello** messages to UDP port 646 at the all-routers multicast address 224.0.0.2. Hellos carry the router's **LSR ID** and its **transport address**, the address it wants to use for the session. Neighbours that hear each other's Hellos are discovered.
2. **Session.** The two routers open **one TCP connection to port 646** between their transport addresses. The router with the higher address takes the active role and opens the connection. They exchange **Initialization** messages to agree on parameters, then **Keepalives** to keep the session alive.

The transport address is normally the router's **loopback**. A session between loopbacks survives the failure of any single link between two routers that have several, and it matches the LSR ID that appears in labels and troubleshooting output. Because the loopbacks must be reachable before the session can start, LDP always depends on the IGP.

Once the session is up, each router sends **Address** messages listing all its interface addresses (so the neighbour can map IGP next hops to LDP neighbours) and **Label Mapping** messages, one per FEC.

## How labels are distributed

<figure class="diagram">
<svg viewBox="0 0 720 300" style="width:100%;height:auto;max-width:720px" role="img" aria-label="Label distribution for FEC 10.0.0.2/32 along pe1, p1, p2, pe2. pe2 advertises implicit null; p2 allocates label 18 and advertises it to p1; p1 allocates label 22 and advertises it to pe1. Data then flows the other way: pe1 pushes 22, p1 swaps 22 to 18, p2 pops." xmlns="http://www.w3.org/2000/svg">
<style>
.m4b{ font-family:var(--sans),sans-serif; }
.m4b .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.m4b .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.m4b .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.m4b .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.m4b .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.m4b .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.m4b .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.m4b .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.m4b .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.m4b .h{ fill:var(--text); font-size:15px; font-weight:700; }
.m4b .b{ fill:var(--text); font-size:13px; font-weight:700; }
.m4b .s{ fill:var(--text-dim); font-size:13px; }
.m4b .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.m4b .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.m4b .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.m4b .c{ fill:var(--text-faint); font-size:12.5px; }
.m4b .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.m4b .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.m4b .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.m4b .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m4b-a); }
.m4b .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m4b-a); marker-start:url(#m4b-s); }
.m4b .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#m4b-g); }
.m4b .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#m4b-a); }
.m4b .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.m4b .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.m4b .bad{ stroke:var(--text); stroke-width:3; fill:none; }
</style>
<defs><marker id="m4b-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="m4b-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="m4b-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker></defs>
<g class="m4b">
<circle cx="90" cy="80" r="24" class="dark"/>
<text x="90" y="84.5" text-anchor="middle" class="on" style="fill:var(--bg)">pe1</text>
<circle cx="270" cy="80" r="24" class="pnl"/>
<text x="270" y="84.5" text-anchor="middle" class="b">p1</text>
<circle cx="450" cy="80" r="24" class="pnl"/>
<text x="450" y="84.5" text-anchor="middle" class="b">p2</text>
<circle cx="630" cy="80" r="24" class="dark"/>
<text x="630" y="84.5" text-anchor="middle" class="on" style="fill:var(--bg)">pe2</text>
<line x1="114" y1="80" x2="246" y2="80" class="lk"/>
<line x1="294" y1="80" x2="426" y2="80" class="lk"/>
<line x1="474" y1="80" x2="606" y2="80" class="lk"/>
<text x="630" y="36" text-anchor="middle" class="c">owns 10.0.0.2</text>
<path d="M604 118 L476 118" class="ad"/>
<text x="540.0" y="140" text-anchor="middle" class="a">10.0.0.2/32: implicit null</text>
<path d="M424 118 L296 118" class="ad"/>
<text x="360.0" y="140" text-anchor="middle" class="a">10.0.0.2/32: label 18</text>
<path d="M244 118 L116 118" class="ad"/>
<text x="180.0" y="140" text-anchor="middle" class="a">10.0.0.2/32: label 22</text>
<text x="360" y="172" text-anchor="middle" class="c">label mappings flow upstream (from the egress toward the ingress)</text>
<path d="M114 206 L606 206" class="ar"/>
<text x="360" y="196" text-anchor="middle" class="c">data flows downstream</text>
<rect x="18" y="226" width="144" height="40" rx="6" class="acc"/>
<text x="90.0" y="250.5" text-anchor="middle" class="b">push 22</text>
<rect x="198" y="226" width="144" height="40" rx="6" class="acc"/>
<text x="270.0" y="250.5" text-anchor="middle" class="b">swap 22 to 18</text>
<rect x="378" y="226" width="144" height="40" rx="6" class="acc"/>
<text x="450.0" y="250.5" text-anchor="middle" class="b">pop (PHP)</text>
<rect x="558" y="226" width="144" height="40" rx="6" class="pnl"/>
<text x="630.0" y="250.5" text-anchor="middle" class="s">IP lookup</text>
</g>
</svg>
<figcaption>Downstream unsolicited distribution. Each router picks its own label for the FEC and tells its upstream neighbours.</figcaption>
</figure>

Follow FEC `10.0.0.2/32`, pe2's loopback:

1. pe2 owns the prefix. It advertises **implicit null** for it, asking its neighbours to pop (PHP, Chapter 2).
2. p2 allocates a local label, say 18, and advertises "10.0.0.2/32: label 18" to **all** its LDP neighbours.
3. p1 allocates its own label, 22, and advertises it to all its neighbours, including pe1.
4. pe1 now has what it needs. To reach 10.0.0.2, its IGP next hop is p1, and p1 asked for label 22. pe1 pushes 22.

Mappings flow upstream, from the egress toward the ingress; data then flows downstream along the same chain. Each router chooses its own label independently, which is why the label changes at every hop.

### Three behaviours

LDP's behaviour is described by three choices.

<figure class="diagram">
<svg viewBox="0 0 720 230" style="width:100%;height:auto;max-width:720px" role="img" aria-label="Three LDP behaviour choices with the common default highlighted: label distribution can be downstream unsolicited or downstream on demand; label control can be independent or ordered; label retention can be liberal or conservative." xmlns="http://www.w3.org/2000/svg">
<style>
.m4d{ font-family:var(--sans),sans-serif; }
.m4d .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.m4d .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.m4d .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.m4d .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.m4d .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.m4d .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.m4d .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.m4d .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.m4d .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.m4d .h{ fill:var(--text); font-size:15px; font-weight:700; }
.m4d .b{ fill:var(--text); font-size:13px; font-weight:700; }
.m4d .s{ fill:var(--text-dim); font-size:13px; }
.m4d .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.m4d .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.m4d .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.m4d .c{ fill:var(--text-faint); font-size:12.5px; }
.m4d .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.m4d .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.m4d .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.m4d .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m4d-a); }
.m4d .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m4d-a); marker-start:url(#m4d-s); }
.m4d .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#m4d-g); }
.m4d .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#m4d-a); }
.m4d .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.m4d .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.m4d .bad{ stroke:var(--text); stroke-width:3; fill:none; }
</style>
<defs><marker id="m4d-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="m4d-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="m4d-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker></defs>
<g class="m4d">
<text x="395" y="24" text-anchor="middle" class="a">usual default</text>
<text x="605" y="24" text-anchor="middle" class="c">alternative</text>
<rect x="20" y="40" width="180" height="46" rx="7" class="pnl"/>
<text x="110.0" y="67.5" text-anchor="middle" class="s">distribution</text>
<rect x="300" y="40" width="190" height="46" rx="7" class="acc"/>
<text x="395.0" y="67.5" text-anchor="middle" class="b">downstream unsolicited</text>
<rect x="510" y="40" width="190" height="46" rx="7" class="box"/>
<text x="605.0" y="67.5" text-anchor="middle" class="s">downstream on demand</text>
<rect x="20" y="100" width="180" height="46" rx="7" class="pnl"/>
<text x="110.0" y="127.5" text-anchor="middle" class="s">control</text>
<rect x="300" y="100" width="190" height="46" rx="7" class="acc"/>
<text x="395.0" y="127.5" text-anchor="middle" class="b">independent</text>
<rect x="510" y="100" width="190" height="46" rx="7" class="box"/>
<text x="605.0" y="127.5" text-anchor="middle" class="s">ordered</text>
<rect x="20" y="160" width="180" height="46" rx="7" class="pnl"/>
<text x="110.0" y="187.5" text-anchor="middle" class="s">retention</text>
<rect x="300" y="160" width="190" height="46" rx="7" class="acc"/>
<text x="395.0" y="187.5" text-anchor="middle" class="b">liberal</text>
<rect x="510" y="160" width="190" height="46" rx="7" class="box"/>
<text x="605.0" y="187.5" text-anchor="middle" class="s">conservative</text>
</g>
</svg>
<figcaption>LDP's three knobs. Frame-mode MPLS on Ethernet almost always uses downstream unsolicited with liberal retention.</figcaption>
</figure>

- **Label distribution.** With **downstream unsolicited** (DU), a router advertises its labels to every neighbour without being asked. With **downstream on demand** (DoD), it waits for a request. Ethernet networks use DU.
- **Label control.** With **independent** control, a router advertises a label for a FEC as soon as it has a route for it. With **ordered** control, it waits until its own downstream neighbour has given it a label, so an LSP is advertised only once it is complete end to end. Implementations differ; both converge to the same result.
- **Label retention.** With **liberal** retention, a router keeps every label it receives, even from neighbours that are not its next hop. With **conservative** retention, it discards them.

<figure class="diagram">
<svg viewBox="0 0 720 270" style="width:100%;height:auto;max-width:720px" role="img" aria-label="Liberal label retention on p1: it learns a label for FEC 10.0.0.2/32 from every LDP neighbour, label 18 from p2, label 25 from p3 and label 31 from pe1, but only installs the one from p2, its IGP next hop. The others are kept so a change of next hop needs no new signalling." xmlns="http://www.w3.org/2000/svg">
<style>
.m4c{ font-family:var(--sans),sans-serif; }
.m4c .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.m4c .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.m4c .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.m4c .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.m4c .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.m4c .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.m4c .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.m4c .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.m4c .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.m4c .h{ fill:var(--text); font-size:15px; font-weight:700; }
.m4c .b{ fill:var(--text); font-size:13px; font-weight:700; }
.m4c .s{ fill:var(--text-dim); font-size:13px; }
.m4c .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.m4c .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.m4c .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.m4c .c{ fill:var(--text-faint); font-size:12.5px; }
.m4c .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.m4c .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.m4c .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.m4c .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m4c-a); }
.m4c .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m4c-a); marker-start:url(#m4c-s); }
.m4c .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#m4c-g); }
.m4c .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#m4c-a); }
.m4c .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.m4c .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.m4c .bad{ stroke:var(--text); stroke-width:3; fill:none; }
</style>
<defs><marker id="m4c-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="m4c-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="m4c-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker></defs>
<g class="m4c">
<circle cx="170" cy="140" r="30" class="dark"/>
<text x="170" y="144.5" text-anchor="middle" class="on" style="fill:var(--bg)">p1</text>
<circle cx="470" cy="50" r="24" class="pnl"/>
<text x="470" y="54.5" text-anchor="middle" class="b">p2</text>
<line x1="200" y1="140" x2="446" y2="50" class="ar"/>
<rect x="540" y="32" width="170" height="36" rx="6" class="acc"/>
<text x="625.0" y="54.5" text-anchor="middle" class="b">remote label 18</text>
<circle cx="470" cy="140" r="24" class="pnl"/>
<text x="470" y="144.5" text-anchor="middle" class="b">p3</text>
<line x1="200" y1="140" x2="446" y2="140" class="ag"/>
<rect x="540" y="122" width="170" height="36" rx="6" class="pnl"/>
<text x="625.0" y="144.5" text-anchor="middle" class="s">remote label 25</text>
<circle cx="470" cy="230" r="24" class="pnl"/>
<text x="470" y="234.5" text-anchor="middle" class="b">pe1</text>
<line x1="200" y1="140" x2="446" y2="230" class="ag"/>
<rect x="540" y="212" width="170" height="36" rx="6" class="pnl"/>
<text x="625.0" y="234.5" text-anchor="middle" class="s">remote label 31</text>
<text x="170" y="196" text-anchor="middle" class="b">FEC 10.0.0.2/32</text>
<text x="170" y="216" text-anchor="middle" class="c">IGP next hop: p2</text>
<text x="330" y="76" text-anchor="middle" class="a">installed</text>
<text x="310" y="236" text-anchor="middle" class="c">kept, not used</text>
</g>
</svg>
<figcaption>Liberal retention keeps every binding. When IS-IS moves the next hop to p3, p1 already has p3's label and switches instantly.</figcaption>
</figure>

Liberal retention costs a little memory and buys fast convergence. When IS-IS moves a route from p2 to p3, p1 already holds p3's label for the FEC and simply starts using it; no LDP messages are needed.

### Which prefixes get labels

By default, many LDP implementations allocate a label for every prefix in the routing table, including every link subnet. Only the loopbacks are ever used as BGP next hops, so the rest is wasted state. FRR's `label local allocate host-routes` restricts allocation to /32 host routes, which in a typical core means exactly the loopbacks. The labs use it.

## LDP and the IGP must agree

LDP follows the IGP, but it reacts more slowly. When a link comes up, the IGP may start using it before the LDP session across it is established.

<figure class="diagram">
<svg viewBox="0 0 720 350" style="width:100%;height:auto;max-width:720px" role="img" aria-label="LDP and IGP synchronisation. When the link p1 to p2 comes back up, IS-IS starts using it at once, but the LDP session over it is not yet established, so p1 has no label from p2 and labelled traffic is dropped or sent unlabelled. With synchronisation, IS-IS advertises the link at maximum metric until LDP is ready, and traffic keeps using the path through p3." xmlns="http://www.w3.org/2000/svg">
<style>
.m4e{ font-family:var(--sans),sans-serif; }
.m4e .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.m4e .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.m4e .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.m4e .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.m4e .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.m4e .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.m4e .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.m4e .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.m4e .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.m4e .h{ fill:var(--text); font-size:15px; font-weight:700; }
.m4e .b{ fill:var(--text); font-size:13px; font-weight:700; }
.m4e .s{ fill:var(--text-dim); font-size:13px; }
.m4e .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.m4e .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.m4e .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.m4e .c{ fill:var(--text-faint); font-size:12.5px; }
.m4e .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.m4e .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.m4e .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.m4e .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m4e-a); }
.m4e .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m4e-a); marker-start:url(#m4e-s); }
.m4e .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#m4e-g); }
.m4e .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#m4e-a); }
.m4e .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.m4e .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.m4e .bad{ stroke:var(--text); stroke-width:3; fill:none; }
</style>
<defs><marker id="m4e-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="m4e-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="m4e-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker></defs>
<g class="m4e">
<text x="20" y="30" text-anchor="start" class="b">without sync: the IGP converges first</text>
<circle cx="80" cy="84" r="24" class="dark"/>
<text x="80" y="88.5" text-anchor="middle" class="on" style="fill:var(--bg)">p1</text>
<circle cx="330" cy="84" r="24" class="pnl"/>
<text x="330" y="88.5" text-anchor="middle" class="b">p2</text>
<circle cx="205" cy="134" r="24" class="pnl"/>
<text x="205" y="138.5" text-anchor="middle" class="b">p3</text>
<line x1="104" y1="84" x2="306" y2="84" class="lk"/>
<line x1="98" y1="100" x2="186" y2="122" class="lk"/>
<line x1="224" y1="122" x2="312" y2="100" class="lk"/>
<path d="M106 70 L304 70" class="ar"/>
<text x="205" y="62" text-anchor="middle" class="c">IGP path, no label</text>
<rect x="400" y="62" width="300" height="44" rx="7" class="pnl"/>
<text x="550.0" y="88.5" text-anchor="middle" class="s">no label from p2 yet: VPN traffic lost</text>
<text x="20" y="190" text-anchor="start" class="b">with sync: the IGP waits for LDP</text>
<circle cx="80" cy="244" r="24" class="dark"/>
<text x="80" y="248.5" text-anchor="middle" class="on" style="fill:var(--bg)">p1</text>
<circle cx="330" cy="244" r="24" class="pnl"/>
<text x="330" y="248.5" text-anchor="middle" class="b">p2</text>
<circle cx="205" cy="294" r="24" class="pnl"/>
<text x="205" y="298.5" text-anchor="middle" class="b">p3</text>
<line x1="104" y1="244" x2="306" y2="244" class="lk"/>
<line x1="98" y1="260" x2="186" y2="282" class="lk"/>
<line x1="224" y1="282" x2="312" y2="260" class="lk"/>
<text x="205" y="234" text-anchor="middle" class="c">max metric</text>
<path d="M70 270 Q 205 364 340 270" class="ar"/>
<text x="205" y="338" text-anchor="middle" class="c">traffic stays on the labelled path</text>
<rect x="400" y="222" width="300" height="44" rx="7" class="acc"/>
<text x="550.0" y="248.5" text-anchor="middle" class="b">link at max metric until LDP is up</text>
</g>
</svg>
<figcaption>A link should carry traffic only once it can carry labels. LDP-IGP synchronisation enforces that.</figcaption>
</figure>

For that brief window the router has an IGP next hop but no label from it. Plain IP traffic still works, but labelled traffic, including all VPN traffic, is either sent without the label it needs or dropped. **LDP-IGP synchronisation** fixes this: the IGP advertises the link with maximum metric until LDP is up on it, so traffic avoids the link until it can carry labels. In FRR's IS-IS it is enabled with `mpls ldp-sync` under `router isis`.

A related feature, **LDP session protection**, keeps a targeted LDP session (and its labels) alive across an alternative path when the direct link between two neighbours fails, so labels do not have to be relearned when the link returns.

## Hands on: an LDP core

The lab `labs/lab2-ldp` builds the five-router core from Chapter 3 with IS-IS and LDP. The LDP part of each router's configuration is short:

```text
mpls ldp
 router-id 10.0.0.11
 !
 address-family ipv4
  discovery transport-address 10.0.0.11
  label local allocate host-routes
  !
  interface eth1
  exit
  interface eth2
  exit
  interface eth3
  exit
 exit-address-family
exit
```

Deploy it and start with the neighbours:

```bash
cd labs/lab2-ldp
sudo containerlab deploy -t lab2-ldp.clab.yml
docker exec -it clab-lab2-ldp-p1 vtysh
```

```text
p1# show mpls ldp discovery          ! Hellos heard on each interface
p1# show mpls ldp neighbor           ! sessions: pe1, p2, p3 in OPERATIONAL state
p1# show mpls ldp binding            ! every FEC: local label, remote labels, which is in use
```

In the binding table, each loopback FEC appears several times, once per neighbour, because of liberal retention. Only the entry whose neighbour is the IGP next hop is marked in use. Notice the remote label from the router that owns the prefix: it is `imp-null`.

Now look at what was installed:

```text
p1# show mpls table                  ! the LFIB: in label, out label, next hop
p1# show ip route 10.0.0.2/32        ! the IS-IS route, now with "label N"
```

On pe1, the route to 10.0.0.2 carries a label to push, and on p2 the entry for that FEC pops (implicit null from pe2). The kernel tables match:

```bash
sudo ip netns exec clab-lab2-ldp-pe1 ip route show 10.0.0.2
sudo ip netns exec clab-lab2-ldp-p1 ip -f mpls route show
```

### Watch it reconverge

Break the shortest path by shutting the p1 to p2 link, and see LDP follow the IGP:

```bash
sudo ip netns exec clab-lab2-ldp-p1 ip link set eth2 down
docker exec clab-lab2-ldp-pe1 vtysh -c "show ip route 10.0.0.2/32"
```

The route from pe1 to pe2 now goes through p3 (pe1, p3, pe2: cost 35), and the label pe1 pushes changes to the label that the new next hop advertised. No LDP signalling was needed for the switch: thanks to liberal retention, the label was already there. Bring the link back with `ip link set eth2 up`.

## Summary

- **LDP** assigns a label to each IGP prefix on each router; LSPs **follow the IGP**.
- Discovery uses **UDP 646 Hellos**; labels travel over one **TCP 646 session** per neighbour, usually between **loopbacks**.
- Mappings flow **upstream** from the egress; each router chooses its own label. The egress advertises **implicit null** for PHP.
- Typical behaviour: **downstream unsolicited**, **liberal retention**; restrict allocation to **host routes** to save state.
- Use **LDP-IGP synchronisation** so links carry traffic only once they can carry labels.
