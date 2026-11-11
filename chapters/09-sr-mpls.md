# Chapter 9: Segment Routing with MPLS

LDP works, but it is one more protocol to run, troubleshoot, and keep synchronised with the IGP, and it can only follow the IGP's shortest paths. **Segment Routing (SR)** removes LDP entirely. The IGP itself advertises a label for every router, and the ingress router can steer a packet along almost any path by pushing a list of them. This chapter covers SR on the MPLS data plane, called **SR-MPLS**; Chapter 12 covers the IPv6 variant.

## Source routing, revisited

The idea of segment routing is old: the sender writes the path into the packet. What makes SR practical is that a path is written as a short list of **segments**, each an instruction such as "go to router p3" or "leave through this link", and that most paths need only one or two of them.

<figure class="diagram">
<svg viewBox="0 0 720 320" style="width:100%;height:auto;max-width:720px" role="img" aria-label="Segment routing as source routing: pe1 encodes the path to pe2 through p3 as two segments, the node SID of p3 (16013) and the node SID of pe2 (16002). The core keeps no per-path state; each router only knows how to reach each node SID." xmlns="http://www.w3.org/2000/svg">
<style>
.m9a{ font-family:var(--sans),sans-serif; }
.m9a .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.m9a .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.m9a .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.m9a .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.m9a .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.m9a .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.m9a .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.m9a .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.m9a .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.m9a .h{ fill:var(--text); font-size:15px; font-weight:700; }
.m9a .b{ fill:var(--text); font-size:13px; font-weight:700; }
.m9a .s{ fill:var(--text-dim); font-size:13px; }
.m9a .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.m9a .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.m9a .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.m9a .c{ fill:var(--text-faint); font-size:12.5px; }
.m9a .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.m9a .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.m9a .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.m9a .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m9a-a); }
.m9a .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m9a-a); marker-start:url(#m9a-s); }
.m9a .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#m9a-g); }
.m9a .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#m9a-a); }
.m9a .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.m9a .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.m9a .bad{ stroke:var(--text); stroke-width:3; fill:none; }
</style>
<defs><marker id="m9a-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="m9a-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="m9a-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker></defs>
<g class="m9a">
<line x1="114.0" y1="80.0" x2="246.0" y2="80.0" class="lk"/>
<line x1="294.0" y1="80.0" x2="426.0" y2="80.0" class="lk"/>
<line x1="474.0" y1="80.0" x2="606.0" y2="80.0" class="lk"/>
<line x1="111.6" y1="90.4" x2="338.4" y2="199.6" class="out"/>
<line x1="381.6" y1="199.6" x2="608.4" y2="90.4" class="out"/>
<line x1="283.7" y1="99.7" x2="346.3" y2="190.3" class="lk"/>
<circle cx="90" cy="80" r="24" class="dark"/>
<text x="90" y="84.5" text-anchor="middle" class="on" style="fill:var(--bg)">pe1</text>
<circle cx="270" cy="80" r="24" class="pnl"/>
<text x="270" y="84.5" text-anchor="middle" class="b">p1</text>
<circle cx="450" cy="80" r="24" class="pnl"/>
<text x="450" y="84.5" text-anchor="middle" class="b">p2</text>
<circle cx="630" cy="80" r="24" class="dark"/>
<text x="630" y="84.5" text-anchor="middle" class="on" style="fill:var(--bg)">pe2</text>
<circle cx="360" cy="210" r="24" class="pnl"/>
<text x="360" y="214.5" text-anchor="middle" class="b">p3</text>
<rect x="40" y="250" width="70" height="28" rx="3" class="acc"/>
<text x="75.0" y="268.5" text-anchor="middle" class="m">16013</text>
<rect x="110" y="250" width="70" height="28" rx="3" class="acc"/>
<text x="145.0" y="268.5" text-anchor="middle" class="m">16002</text>
<rect x="180" y="250" width="90" height="28" rx="3" class="box"/>
<text x="225.0" y="268.5" text-anchor="middle" class="m">payload</text>
<text x="155" y="302" text-anchor="middle" class="c">segment list pushed by pe1</text>
<rect x="400" y="240" width="310" height="50" rx="7" class="pnl"/>
<text x="555.0" y="260.5" text-anchor="middle" class="b">go to p3, then go to pe2</text>
<text x="555.0" y="278.5" text-anchor="middle" class="c">each segment is one instruction</text>
<text x="565" y="172" text-anchor="middle" class="a">p3 to pe2 (16002)</text>
<text x="165" y="160" text-anchor="middle" class="a">pe1 to p3 (16013)</text>
</g>
</svg>
<figcaption>The path lives in the packet. Routers in the middle hold one entry per node, not one per path.</figcaption>
</figure>

Each segment is identified by a **SID** (segment identifier). On the MPLS data plane a SID is simply a label, and a list of segments is a label stack. Nothing about forwarding changes: routers still push, swap, and pop. What changes is where labels come from and how much state the core keeps. With LDP, every router holds a label binding for every FEC from every neighbour. With SR, every router holds one entry per node in the network, learned from the IGP, and any path is built at the ingress by combining them. Paths themselves create no state in the core.

## Two kinds of segment

<figure class="diagram">
<svg viewBox="0 0 720 270" style="width:100%;height:auto;max-width:720px" role="img" aria-label="Two kinds of segment: a prefix SID such as a node SID is global, advertised by its owner with an index in IS-IS, and means the shortest path to that node from anywhere; an adjacency SID is local to one router and means send out of this specific link." xmlns="http://www.w3.org/2000/svg">
<style>
.m9b{ font-family:var(--sans),sans-serif; }
.m9b .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.m9b .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.m9b .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.m9b .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.m9b .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.m9b .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.m9b .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.m9b .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.m9b .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.m9b .h{ fill:var(--text); font-size:15px; font-weight:700; }
.m9b .b{ fill:var(--text); font-size:13px; font-weight:700; }
.m9b .s{ fill:var(--text-dim); font-size:13px; }
.m9b .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.m9b .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.m9b .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.m9b .c{ fill:var(--text-faint); font-size:12.5px; }
.m9b .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.m9b .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.m9b .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.m9b .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m9b-a); }
.m9b .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m9b-a); marker-start:url(#m9b-s); }
.m9b .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#m9b-g); }
.m9b .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#m9b-a); }
.m9b .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.m9b .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.m9b .bad{ stroke:var(--text); stroke-width:3; fill:none; }
</style>
<defs><marker id="m9b-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="m9b-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="m9b-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker></defs>
<g class="m9b">
<rect x="20" y="20" width="330" height="120" rx="7" class="acc"/>
<text x="185.0" y="51.5" text-anchor="middle" class="b">prefix SID (node SID)</text>
<text x="185.0" y="73.5" text-anchor="middle" class="s">global: index 2 on pe2's loopback</text>
<text x="185.0" y="95.5" text-anchor="middle" class="m">label = SRGB base + index = 16002</text>
<text x="185.0" y="117.5" text-anchor="middle" class="s">means: shortest path to pe2</text>
<rect x="370" y="20" width="330" height="120" rx="7" class="pnl"/>
<text x="535.0" y="51.5" text-anchor="middle" class="b">adjacency SID</text>
<text x="535.0" y="73.5" text-anchor="middle" class="s">local: allocated per link by p1</text>
<text x="535.0" y="95.5" text-anchor="middle" class="m">label e.g. 15001 (dynamic)</text>
<text x="535.0" y="117.5" text-anchor="middle" class="c">means: send out of link p1 to p3</text>
<rect x="20" y="170" width="680" height="76" rx="7" class="box"/>
<text x="360.0" y="200.5" text-anchor="middle" class="s">Both are advertised in IS-IS (or OSPF) sub-TLVs. No LDP, no RSVP.</text>
<text x="360.0" y="224.5" text-anchor="middle" class="m">A segment list mixes them: [16013, 15001, 16002]</text>
</g>
</svg>
<figcaption>Global prefix segments follow the IGP's shortest paths; local adjacency segments pin a specific link.</figcaption>
</figure>

- A **prefix SID** is attached to a prefix and is **global**: every router in the domain understands it. The most important is the **node SID**, a prefix SID on a router's loopback. It means "the shortest path to this router", including ECMP. It is configured on the owner and flooded in IS-IS or OSPF.
- An **adjacency SID** is attached to one of a router's links and is **local** to that router. It means "send out of this exact link", whatever the IGP's shortest path. Routers allocate adjacency SIDs automatically and advertise them in the IGP as well.

Prefix and adjacency SIDs combine freely. `[16013, 16002]` means "go to p3 by the shortest path, then to pe2 by the shortest path". Adding an adjacency SID pins one particular link in the middle.

## The SRGB and the index

A prefix SID is advertised as an **index**, not a label. Every router reserves a block of labels for SR, the **Segment Routing Global Block (SRGB)**, and the label for a prefix SID is the SRGB base plus the index.

<figure class="diagram">
<svg viewBox="0 0 720 230" style="width:100%;height:auto;max-width:720px" role="img" aria-label="The SRGB and index: every router reserves the same label block, 16000 to 23999, so node SID index 2 becomes label 16002 on every router. If one router used a different block, for example 20000, its label for index 2 would be 20002 and its neighbours would use that value when sending to it." xmlns="http://www.w3.org/2000/svg">
<style>
.m9c{ font-family:var(--sans),sans-serif; }
.m9c .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.m9c .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.m9c .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.m9c .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.m9c .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.m9c .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.m9c .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.m9c .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.m9c .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.m9c .h{ fill:var(--text); font-size:15px; font-weight:700; }
.m9c .b{ fill:var(--text); font-size:13px; font-weight:700; }
.m9c .s{ fill:var(--text-dim); font-size:13px; }
.m9c .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.m9c .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.m9c .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.m9c .c{ fill:var(--text-faint); font-size:12.5px; }
.m9c .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.m9c .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.m9c .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.m9c .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m9c-a); }
.m9c .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m9c-a); marker-start:url(#m9c-s); }
.m9c .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#m9c-g); }
.m9c .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#m9c-a); }
.m9c .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.m9c .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.m9c .bad{ stroke:var(--text); stroke-width:3; fill:none; }
</style>
<defs><marker id="m9c-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="m9c-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="m9c-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker></defs>
<g class="m9c">
<rect x="30" y="30" width="150" height="80" rx="7" class="acc"/>
<text x="105.0" y="53.5" text-anchor="middle" class="b">pe1</text>
<text x="105.0" y="74.5" text-anchor="middle" class="m">SRGB 16000</text>
<text x="105.0" y="95.5" text-anchor="middle" class="m">index 2 = 16002</text>
<rect x="202" y="30" width="150" height="80" rx="7" class="acc"/>
<text x="277.0" y="53.5" text-anchor="middle" class="b">p1</text>
<text x="277.0" y="74.5" text-anchor="middle" class="m">SRGB 16000</text>
<text x="277.0" y="95.5" text-anchor="middle" class="m">index 2 = 16002</text>
<rect x="374" y="30" width="150" height="80" rx="7" class="acc"/>
<text x="449.0" y="53.5" text-anchor="middle" class="b">p2</text>
<text x="449.0" y="74.5" text-anchor="middle" class="m">SRGB 16000</text>
<text x="449.0" y="95.5" text-anchor="middle" class="m">index 2 = 16002</text>
<rect x="546" y="30" width="150" height="80" rx="7" class="pnl"/>
<text x="621.0" y="53.5" text-anchor="middle" class="b">pe2</text>
<text x="621.0" y="74.5" text-anchor="middle" class="m">SRGB 16000</text>
<text x="621.0" y="95.5" text-anchor="middle" class="ma">index 2 = 16002</text>
<rect x="30" y="140" width="660" height="64" rx="7" class="pnl"/>
<text x="360.0" y="165.5" text-anchor="middle" class="s">Same SRGB everywhere: the label for a node never changes along the path.</text>
<text x="360.0" y="187.5" text-anchor="middle" class="c">Different SRGBs still work: a router uses its neighbour's base + index.</text>
</g>
</svg>
<figcaption>Index plus base gives the label. Using one SRGB network-wide makes labels readable and troubleshooting easy.</figcaption>
</figure>

If every router uses the same SRGB, and best practice says they should, the label for a node is the same on every router: pe2 with index 2 is label 16002 everywhere in the lab. That makes label stacks readable at a glance. The standard still works if SRGBs differ, because a router always uses its **neighbour's** base when sending to that neighbour; labels then change hop by hop as they do in LDP.

SR-MPLS also keeps PHP. By default, the owner of a node SID asks its neighbours to pop it (the **no-PHP** flag is off). It can request explicit null instead, for the same reasons as in LDP.

## Forwarding on a node SID

<figure class="diagram">
<svg viewBox="0 0 720 230" style="width:100%;height:auto;max-width:720px" role="img" aria-label="Forwarding on node SID 16002 from pe1 to pe2 along the shortest path: pe1 pushes 16002, p1 swaps 16002 for 16002, p2 pops it because pe2 asked for PHP, and pe2 receives the payload." xmlns="http://www.w3.org/2000/svg">
<style>
.m9d{ font-family:var(--sans),sans-serif; }
.m9d .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.m9d .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.m9d .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.m9d .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.m9d .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.m9d .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.m9d .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.m9d .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.m9d .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.m9d .h{ fill:var(--text); font-size:15px; font-weight:700; }
.m9d .b{ fill:var(--text); font-size:13px; font-weight:700; }
.m9d .s{ fill:var(--text-dim); font-size:13px; }
.m9d .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.m9d .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.m9d .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.m9d .c{ fill:var(--text-faint); font-size:12.5px; }
.m9d .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.m9d .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.m9d .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.m9d .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m9d-a); }
.m9d .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m9d-a); marker-start:url(#m9d-s); }
.m9d .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#m9d-g); }
.m9d .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#m9d-a); }
.m9d .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.m9d .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.m9d .bad{ stroke:var(--text); stroke-width:3; fill:none; }
</style>
<defs><marker id="m9d-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="m9d-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="m9d-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker></defs>
<g class="m9d">
<circle cx="90" cy="60" r="24" class="dark"/>
<text x="90" y="64.5" text-anchor="middle" class="on" style="fill:var(--bg)">pe1</text>
<circle cx="270" cy="60" r="24" class="pnl"/>
<text x="270" y="64.5" text-anchor="middle" class="b">p1</text>
<circle cx="450" cy="60" r="24" class="pnl"/>
<text x="450" y="64.5" text-anchor="middle" class="b">p2</text>
<circle cx="630" cy="60" r="24" class="dark"/>
<text x="630" y="64.5" text-anchor="middle" class="on" style="fill:var(--bg)">pe2</text>
<line x1="114" y1="60" x2="246" y2="60" class="lk"/>
<line x1="294" y1="60" x2="426" y2="60" class="lk"/>
<line x1="474" y1="60" x2="606" y2="60" class="lk"/>
<rect x="145" y="100" width="60" height="28" rx="3" class="acc"/>
<text x="175.0" y="118.5" text-anchor="middle" class="m">16002</text>
<rect x="205" y="100" width="40" height="28" rx="3" class="box"/>
<text x="225.0" y="118.5" text-anchor="middle" class="m">IP</text>
<rect x="325" y="100" width="60" height="28" rx="3" class="acc"/>
<text x="355.0" y="118.5" text-anchor="middle" class="m">16002</text>
<rect x="385" y="100" width="40" height="28" rx="3" class="box"/>
<text x="405.0" y="118.5" text-anchor="middle" class="m">IP</text>
<rect x="520" y="100" width="40" height="28" rx="3" class="box"/>
<text x="540.0" y="118.5" text-anchor="middle" class="m">IP</text>
<text x="90" y="158" text-anchor="middle" class="a">push 16002</text>
<text x="270" y="158" text-anchor="middle" class="a">swap 16002 to 16002</text>
<text x="450" y="158" text-anchor="middle" class="a">pop (PHP)</text>
<text x="360" y="206" text-anchor="middle" class="c">the same label value at every hop: the label names the destination, not the hop</text>
</g>
</svg>
<figcaption>A node SID behaves like an LDP LSP whose label is the same everywhere.</figcaption>
</figure>

A packet labelled 16002 follows the IGP's shortest path to pe2. Each router looks up 16002 in its label table, which the IGP filled in, and forwards it toward pe2, swapping 16002 for 16002 (or for the neighbour's equivalent). The router before pe2 pops it. This is exactly what an LDP LSP does, without LDP.

## Why operators move from LDP to SR

<figure class="diagram">
<svg viewBox="0 0 720 270" style="width:100%;height:auto;max-width:720px" role="img" aria-label="LDP versus SR-MPLS: LDP needs the IGP plus a separate protocol with sessions, and chooses labels per hop; SR-MPLS needs only IGP extensions, uses global labels, supports explicit paths without signalling, and gives TI-LFA full fast reroute coverage." xmlns="http://www.w3.org/2000/svg">
<style>
.m9e{ font-family:var(--sans),sans-serif; }
.m9e .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.m9e .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.m9e .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.m9e .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.m9e .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.m9e .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.m9e .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.m9e .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.m9e .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.m9e .h{ fill:var(--text); font-size:15px; font-weight:700; }
.m9e .b{ fill:var(--text); font-size:13px; font-weight:700; }
.m9e .s{ fill:var(--text-dim); font-size:13px; }
.m9e .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.m9e .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.m9e .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.m9e .c{ fill:var(--text-faint); font-size:12.5px; }
.m9e .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.m9e .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.m9e .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.m9e .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m9e-a); }
.m9e .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m9e-a); marker-start:url(#m9e-s); }
.m9e .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#m9e-g); }
.m9e .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#m9e-a); }
.m9e .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.m9e .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.m9e .bad{ stroke:var(--text); stroke-width:3; fill:none; }
</style>
<defs><marker id="m9e-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="m9e-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="m9e-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker></defs>
<g class="m9e">
<text x="330" y="24" text-anchor="middle" class="b">LDP</text>
<text x="570" y="24" text-anchor="middle" class="a">SR-MPLS</text>
<text x="20" y="61" text-anchor="start" class="s">protocols</text>
<rect x="210" y="36" width="240" height="36" rx="5" class="pnl"/>
<text x="330.0" y="58.5" text-anchor="middle" class="s">IGP + LDP sessions</text>
<rect x="460" y="36" width="240" height="36" rx="5" class="acc"/>
<text x="580.0" y="58.5" text-anchor="middle" class="b">IGP extensions only</text>
<text x="20" y="105" text-anchor="start" class="s">labels</text>
<rect x="210" y="80" width="240" height="36" rx="5" class="pnl"/>
<text x="330.0" y="102.5" text-anchor="middle" class="s">local, different at each hop</text>
<rect x="460" y="80" width="240" height="36" rx="5" class="acc"/>
<text x="580.0" y="102.5" text-anchor="middle" class="b">global (SRGB + index)</text>
<text x="20" y="149" text-anchor="start" class="s">explicit paths</text>
<rect x="210" y="124" width="240" height="36" rx="5" class="pnl"/>
<text x="330.0" y="146.5" text-anchor="middle" class="s">needs RSVP-TE</text>
<rect x="460" y="124" width="240" height="36" rx="5" class="acc"/>
<text x="580.0" y="146.5" text-anchor="middle" class="b">segment list at the ingress</text>
<text x="20" y="193" text-anchor="start" class="s">fast reroute</text>
<rect x="210" y="168" width="240" height="36" rx="5" class="pnl"/>
<text x="330.0" y="190.5" text-anchor="middle" class="s">LFA / remote LFA</text>
<rect x="460" y="168" width="240" height="36" rx="5" class="acc"/>
<text x="580.0" y="190.5" text-anchor="middle" class="b">TI-LFA, full coverage</text>
<text x="20" y="237" text-anchor="start" class="s">IGP sync issues</text>
<rect x="210" y="212" width="240" height="36" rx="5" class="pnl"/>
<text x="330.0" y="234.5" text-anchor="middle" class="s">yes</text>
<rect x="460" y="212" width="240" height="36" rx="5" class="acc"/>
<text x="580.0" y="234.5" text-anchor="middle" class="b">none</text>
</g>
</svg>
<figcaption>Why SR-MPLS is replacing LDP in new networks: fewer protocols, less state, more capability.</figcaption>
</figure>

- **One protocol fewer.** No LDP sessions, no LDP-IGP synchronisation problems, no label bindings to compare between protocols.
- **Global, predictable labels.** The label for a router is known in advance from its index, which helps both automation and troubleshooting.
- **Traffic engineering without signalling.** An explicit path is a label stack at the ingress (Chapter 11); RSVP-TE's per-tunnel state in the core disappears.
- **Full fast reroute coverage.** TI-LFA (Chapter 10) protects against any single link or node failure, in any topology.

### Living with LDP

Networks rarely switch overnight. SR and LDP can run side by side, and an ingress can prefer one or the other. Where an SR domain meets an LDP-only domain, border routers that run both stitch SR labels to LDP labels.

<figure class="diagram">
<svg viewBox="0 0 720 230" style="width:100%;height:auto;max-width:720px" role="img" aria-label="SR and LDP interworking: an SR domain and an LDP domain meet at border routers that run both. A mapping server advertises SIDs on behalf of LDP-only routers so SR routers can reach them, and border routers stitch SR labels to LDP labels." xmlns="http://www.w3.org/2000/svg">
<style>
.m9f{ font-family:var(--sans),sans-serif; }
.m9f .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.m9f .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.m9f .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.m9f .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.m9f .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.m9f .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.m9f .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.m9f .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.m9f .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.m9f .h{ fill:var(--text); font-size:15px; font-weight:700; }
.m9f .b{ fill:var(--text); font-size:13px; font-weight:700; }
.m9f .s{ fill:var(--text-dim); font-size:13px; }
.m9f .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.m9f .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.m9f .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.m9f .c{ fill:var(--text-faint); font-size:12.5px; }
.m9f .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.m9f .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.m9f .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.m9f .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m9f-a); }
.m9f .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m9f-a); marker-start:url(#m9f-s); }
.m9f .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#m9f-g); }
.m9f .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#m9f-a); }
.m9f .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.m9f .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.m9f .bad{ stroke:var(--text); stroke-width:3; fill:none; }
</style>
<defs><marker id="m9f-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="m9f-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="m9f-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker></defs>
<g class="m9f">
<rect x="20" y="30" width="330" height="120" rx="7" class="zone"/>
<text x="185" y="52" text-anchor="middle" class="a">SR domain</text>
<rect x="370" y="30" width="330" height="120" rx="7" class="zone"/>
<text x="535" y="52" text-anchor="middle" class="s">LDP domain</text>
<circle cx="80" cy="100" r="24" class="dark"/>
<text x="80" y="104.5" text-anchor="middle" class="on" style="fill:var(--bg)">pe1</text>
<circle cx="200" cy="100" r="24" class="pnl"/>
<text x="200" y="104.5" text-anchor="middle" class="b">p</text>
<circle cx="360" cy="100" r="24" class="acc"/>
<text x="360" y="104.5" text-anchor="middle" class="b">BR</text>
<circle cx="510" cy="100" r="24" class="pnl"/>
<text x="510" y="104.5" text-anchor="middle" class="b">p</text>
<circle cx="640" cy="100" r="24" class="dark"/>
<text x="640" y="104.5" text-anchor="middle" class="on" style="fill:var(--bg)">pe9</text>
<line x1="104" y1="100" x2="176" y2="100" class="lk"/>
<line x1="224" y1="100" x2="336" y2="100" class="lk"/>
<line x1="384" y1="100" x2="486" y2="100" class="lk"/>
<line x1="534" y1="100" x2="616" y2="100" class="lk"/>
<rect x="140" y="170" width="440" height="46" rx="7" class="pnl"/>
<text x="360.0" y="197.5" text-anchor="middle" class="s">mapping server: index 9 for 10.0.0.9/32 (LDP-only pe9)</text>
</g>
</svg>
<figcaption>Migration happens in steps. Border routers translate between SR labels and LDP labels while both exist.</figcaption>
</figure>

LDP-only routers cannot advertise their own prefix SIDs, so a **mapping server** advertises SIDs on their behalf, letting SR routers reach them with node SIDs. Migration then proceeds router by router.

## Configuring SR-MPLS in FRR

In FRR, SR is part of the IGP configuration. For IS-IS:

```text
router isis CORE
 net 49.0001.0000.0000.0001.00
 is-type level-2-only
 metric-style wide
 segment-routing on
 segment-routing global-block 16000 23999
 segment-routing node-msd 8
 segment-routing prefix 10.0.0.1/32 index 1
exit
```

- `segment-routing on` enables SR in IS-IS.
- `global-block` sets the SRGB; use the same range everywhere.
- `node-msd` advertises the maximum number of labels this router can push (its maximum SID depth), which matters for traffic engineering.
- `segment-routing prefix ... index` assigns the node SID. Options such as `no-php-flag` and `explicit-null` change the PHP behaviour.

Adjacency SIDs are allocated automatically for every IS-IS adjacency. OSPF has equivalent commands under `router ospf`, together with `capability opaque` and `router-info area`.

## Hands on: L3VPN over SR-MPLS

The lab `labs/lab5-sr-mpls` is the L3VPN network from Chapter 6 with one change: **LDP is gone**. IS-IS carries the SIDs, and the VPN traffic rides on node SIDs. The BGP and VRF configuration is identical to Chapter 6, which shows how cleanly services separate from transport. (The interfaces also enable TI-LFA, used in Chapter 10.)

```bash
cd labs/lab5-sr-mpls
sudo containerlab deploy -t lab5-sr-mpls.clab.yml
docker exec -it clab-lab5-sr-mpls-pe1 vtysh
```

Explore the SR state:

```text
pe1# show isis segment-routing node      ! every node: SRGB, MSD, node SIDs
pe1# show mpls table                     ! 16001 to 16013, plus adjacency SIDs
pe1# show ip route 10.0.0.2/32           ! the IS-IS route, label 16002
```

The MPLS table on every router contains the same 16000-range entries, one per node, plus a handful of adjacency SIDs from the dynamic range. The VPN routes resolve exactly as before, but now the transport label is 16002:

```text
pe1# show ip route vrf RED 192.168.2.0/24
```

The route has two labels: **16002** (transport, a node SID) and pe2's VPN label. Test the customers, then look at the wire:

```bash
docker exec clab-lab5-sr-mpls-ce1 ping -c 3 -I 192.168.1.1 192.168.2.1
sudo ip netns exec clab-lab5-sr-mpls-p1 tcpdump -nni eth2 mpls
```

On p1's link toward p2 you will see label 16002 above the VPN label: the same value pe1 pushed.

### Experiment: an explicit path by hand

SR lets you write a path into a packet without any signalling. On pe1, add a static route toward one of p2's addresses that sends the packet to p3 first, carrying p2's node SID:

```text
pe1# configure terminal
pe1(config)# ip route 10.10.3.1/32 10.10.4.2 label 16012
pe1(config)# end
pe1# show ip route 10.10.3.1/32
```

`10.10.4.2` is p3's address on the pe1 to p3 link, and `10.10.3.1` is p2's address on its link to pe2. pe1 hands the packet directly to p3 with label 16012, "go to p2". p3 forwards it along its own shortest path to p2, which runs through p1. The packet therefore travels pe1, p3, p1, p2, a path the IGP would never choose from pe1, and no router along it was configured for this. Capture on p3's `eth1` while pinging `10.10.3.1` from pe1 (`ping -I 10.0.0.1 10.10.3.1` in pe1's shell) to confirm the detour. Chapter 11 turns this idea into managed SR policies.

## Summary

- **Segment Routing** writes the path into the packet as a list of **segments**; on MPLS each SID is a **label**.
- **Prefix (node) SIDs** are global and follow shortest paths; **adjacency SIDs** are local and pin a link.
- The label for a prefix SID is **SRGB base + index**; use one **SRGB** network-wide.
- The IGP (**IS-IS** or **OSPF**) carries SIDs; **no LDP** is needed, and services such as L3VPN work unchanged.
- SR and LDP coexist during migration through **stitching** and a **mapping server**.
