# Chapter 1: Why MPLS

IP routing is simple and robust: every router looks at a packet's destination address, finds the longest matching prefix in its table, and sends the packet one hop closer. Yet almost every large network adds another layer on top of it. This chapter explains what that layer is, what problem it originally solved, and why it is still everywhere decades later.

## Forwarding on labels

**Multiprotocol Label Switching (MPLS)** forwards packets on a short, fixed-length **label** instead of the destination address. The first router to receive a packet looks at it carefully, decides how it should cross the network, and attaches a label. Every router after that looks only at the label.

<figure class="diagram">
<svg viewBox="0 0 720 300" style="width:100%;height:auto;max-width:720px" role="img" aria-label="Top row: in plain IP routing every router does a longest-prefix-match lookup on the destination address. Bottom row: with MPLS only the ingress router classifies the packet; the routers in the middle do an exact-match lookup on a short label and swap it." xmlns="http://www.w3.org/2000/svg">
<style>
.m1a{ font-family:var(--sans),sans-serif; }
.m1a .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.m1a .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.m1a .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.m1a .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.m1a .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.m1a .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.m1a .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.m1a .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.m1a .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.m1a .h{ fill:var(--text); font-size:15px; font-weight:700; }
.m1a .b{ fill:var(--text); font-size:13px; font-weight:700; }
.m1a .s{ fill:var(--text-dim); font-size:13px; }
.m1a .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.m1a .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.m1a .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.m1a .c{ fill:var(--text-faint); font-size:12.5px; }
.m1a .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.m1a .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.m1a .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.m1a .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m1a-a); }
.m1a .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m1a-a); marker-start:url(#m1a-s); }
.m1a .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#m1a-g); }
.m1a .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#m1a-a); }
.m1a .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.m1a .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.m1a .bad{ stroke:var(--text); stroke-width:3; fill:none; }
</style>
<defs><marker id="m1a-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="m1a-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="m1a-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker></defs>
<g class="m1a">
<text x="20" y="24" text-anchor="start" class="h">Plain IP routing</text>
<circle cx="60" cy="70" r="24" class="pnl"/>
<text x="60" y="74.5" text-anchor="middle" class="b">R1</text>
<line x1="84" y1="70" x2="236" y2="70" class="ar"/>
<circle cx="260" cy="70" r="24" class="pnl"/>
<text x="260" y="74.5" text-anchor="middle" class="b">R2</text>
<line x1="284" y1="70" x2="436" y2="70" class="ar"/>
<circle cx="460" cy="70" r="24" class="pnl"/>
<text x="460" y="74.5" text-anchor="middle" class="b">R3</text>
<line x1="484" y1="70" x2="636" y2="70" class="ar"/>
<circle cx="660" cy="70" r="24" class="pnl"/>
<text x="660" y="74.5" text-anchor="middle" class="b">R4</text>
<text x="60" y="120" text-anchor="middle" class="c">longest-prefix</text>
<text x="60" y="136" text-anchor="middle" class="c">match on dst IP</text>
<text x="260" y="120" text-anchor="middle" class="c">longest-prefix</text>
<text x="260" y="136" text-anchor="middle" class="c">match on dst IP</text>
<text x="460" y="120" text-anchor="middle" class="c">longest-prefix</text>
<text x="460" y="136" text-anchor="middle" class="c">match on dst IP</text>
<text x="660" y="120" text-anchor="middle" class="c">longest-prefix</text>
<text x="660" y="136" text-anchor="middle" class="c">match on dst IP</text>
<text x="20" y="176" text-anchor="start" class="h">MPLS</text>
<circle cx="60" cy="222" r="24" class="dark"/>
<text x="60" y="226.5" text-anchor="middle" class="on" style="fill:var(--bg)">LER</text>
<circle cx="260" cy="222" r="24" class="pnl"/>
<text x="260" y="226.5" text-anchor="middle" class="b">LSR</text>
<circle cx="460" cy="222" r="24" class="pnl"/>
<text x="460" y="226.5" text-anchor="middle" class="b">LSR</text>
<circle cx="660" cy="222" r="24" class="dark"/>
<text x="660" y="226.5" text-anchor="middle" class="on" style="fill:var(--bg)">LER</text>
<line x1="84" y1="222" x2="236" y2="222" class="ar"/>
<rect x="120" y="186" width="80" height="24" rx="5" class="acc"/>
<text x="160.0" y="202.5" text-anchor="middle" class="b">label 21</text>
<line x1="284" y1="222" x2="436" y2="222" class="ar"/>
<rect x="320" y="186" width="80" height="24" rx="5" class="acc"/>
<text x="360.0" y="202.5" text-anchor="middle" class="b">label 37</text>
<line x1="484" y1="222" x2="636" y2="222" class="ar"/>
<rect x="520" y="186" width="80" height="24" rx="5" class="acc"/>
<text x="560.0" y="202.5" text-anchor="middle" class="b">label 18</text>
<text x="60" y="272" text-anchor="middle" class="c">classify once,</text>
<text x="60" y="288" text-anchor="middle" class="c">push a label</text>
<text x="360" y="272" text-anchor="middle" class="c">exact match on a 20-bit label,</text>
<text x="360" y="288" text-anchor="middle" class="c">swap it, forward</text>
<text x="660" y="272" text-anchor="middle" class="c">pop the label,</text>
<text x="660" y="288" text-anchor="middle" class="c">deliver the packet</text>
</g>
</svg>
<figcaption>Classify once at the edge, then switch on labels. The core never looks at the IP header again.</figcaption>
</figure>

Three terms appear in every MPLS discussion:

- A **Label Switching Router (LSR)** is any router that forwards packets by label.
- A **Label Edge Router (LER)** sits at the edge. The ingress LER puts the label on (a **push**), and the egress LER takes it off (a **pop**). Routers in the middle replace one label with another (a **swap**).
- A **Label Switched Path (LSP)** is the path a labelled packet follows from ingress to egress. It is always **one-directional**; traffic in the other direction uses a separate LSP.

The decision at the ingress is called classifying the packet into a **Forwarding Equivalence Class (FEC)**: a group of packets that should all be treated the same way. The simplest FEC is "everything going to the same egress router", but a FEC can also be "this customer's traffic" or "voice traffic toward this site".

## The original motivation, and why it no longer matters

In the mid 1990s, longest-prefix match was expensive. Routers did it in software, while switches could do an exact-match lookup in hardware at wire speed. Several vendors proposed attaching a short tag to each packet so the core could switch instead of route: Cisco's tag switching and Ipsilon's IP switching among them. The IETF merged these ideas into MPLS.

<figure class="diagram">
<svg viewBox="0 0 720 196" style="width:100%;height:auto;max-width:720px" role="img" aria-label="A timeline: tag switching in 1996, the MPLS architecture RFC 3031 in 2001, RSVP-TE RFC 3209 in 2001, BGP/MPLS IP VPNs RFC 4364 in 2006, LDP RFC 5036 in 2007, the Segment Routing architecture RFC 8402 in 2018, and SRv6 network programming RFC 8986 in 2021." xmlns="http://www.w3.org/2000/svg">
<style>
.m1b{ font-family:var(--sans),sans-serif; }
.m1b .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.m1b .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.m1b .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.m1b .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.m1b .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.m1b .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.m1b .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.m1b .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.m1b .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.m1b .h{ fill:var(--text); font-size:15px; font-weight:700; }
.m1b .b{ fill:var(--text); font-size:13px; font-weight:700; }
.m1b .s{ fill:var(--text-dim); font-size:13px; }
.m1b .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.m1b .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.m1b .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.m1b .c{ fill:var(--text-faint); font-size:12.5px; }
.m1b .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.m1b .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.m1b .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.m1b .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m1b-a); }
.m1b .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m1b-a); marker-start:url(#m1b-s); }
.m1b .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#m1b-g); }
.m1b .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#m1b-a); }
.m1b .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.m1b .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.m1b .bad{ stroke:var(--text); stroke-width:3; fill:none; }
</style>
<defs><marker id="m1b-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="m1b-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="m1b-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker></defs>
<g class="m1b">
<line x1="20" y1="98" x2="700" y2="98" class="lk"/>
<circle cx="60" cy="98" r="7" class="acc"/>
<text x="60" y="30" text-anchor="middle" class="b">Tag switching</text>
<text x="60" y="52" text-anchor="middle" class="a">1996</text>
<text x="60" y="74" text-anchor="middle" class="c">Cisco, Ipsilon</text>
<circle cx="160" cy="98" r="7" class="dark"/>
<text x="160" y="128" text-anchor="middle" class="c">MPLS architecture</text>
<text x="160" y="150" text-anchor="middle" class="a">2001</text>
<text x="160" y="172" text-anchor="middle" class="b">RFC 3031</text>
<circle cx="260" cy="98" r="7" class="acc"/>
<text x="260" y="30" text-anchor="middle" class="b">RFC 3209</text>
<text x="260" y="52" text-anchor="middle" class="a">2001</text>
<text x="260" y="74" text-anchor="middle" class="c">RSVP-TE</text>
<circle cx="360" cy="98" r="7" class="acc"/>
<text x="360" y="128" text-anchor="middle" class="c">BGP/MPLS VPNs</text>
<text x="360" y="150" text-anchor="middle" class="a">2006</text>
<text x="360" y="172" text-anchor="middle" class="b">RFC 4364</text>
<circle cx="460" cy="98" r="7" class="acc"/>
<text x="460" y="30" text-anchor="middle" class="b">RFC 5036</text>
<text x="460" y="52" text-anchor="middle" class="a">2007</text>
<text x="460" y="74" text-anchor="middle" class="c">LDP</text>
<circle cx="560" cy="98" r="7" class="dark"/>
<text x="560" y="128" text-anchor="middle" class="c">Segment Routing</text>
<text x="560" y="150" text-anchor="middle" class="a">2018</text>
<text x="560" y="172" text-anchor="middle" class="b">RFC 8402</text>
<circle cx="660" cy="98" r="7" class="dark"/>
<text x="660" y="30" text-anchor="middle" class="b">RFC 8986</text>
<text x="660" y="52" text-anchor="middle" class="a">2021</text>
<text x="660" y="74" text-anchor="middle" class="c">SRv6</text>
</g>
</svg>
<figcaption>Thirty years of label switching. The data plane barely changed; the control planes around it kept evolving.</figcaption>
</figure>

Hardware soon learned to do longest-prefix match at line rate, so the speed argument disappeared. MPLS survived anyway, because a label turned out to be something more useful than a fast lookup key: it is a **tunnel**. Once a packet carries a label, the core forwards it without ever reading the IP header underneath. That one property enables everything else in this book.

## What the tunnel buys you

<figure class="diagram">
<svg viewBox="0 0 720 250" style="width:100%;height:auto;max-width:720px" role="img" aria-label="Label switching in the centre enables four things around it: BGP-free cores, VPNs that keep customers apart, traffic engineering that picks non-shortest paths, and fast reroute that repairs failures in tens of milliseconds." xmlns="http://www.w3.org/2000/svg">
<style>
.m1c{ font-family:var(--sans),sans-serif; }
.m1c .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.m1c .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.m1c .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.m1c .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.m1c .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.m1c .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.m1c .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.m1c .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.m1c .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.m1c .h{ fill:var(--text); font-size:15px; font-weight:700; }
.m1c .b{ fill:var(--text); font-size:13px; font-weight:700; }
.m1c .s{ fill:var(--text-dim); font-size:13px; }
.m1c .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.m1c .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.m1c .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.m1c .c{ fill:var(--text-faint); font-size:12.5px; }
.m1c .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.m1c .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.m1c .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.m1c .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m1c-a); }
.m1c .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m1c-a); marker-start:url(#m1c-s); }
.m1c .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#m1c-g); }
.m1c .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#m1c-a); }
.m1c .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.m1c .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.m1c .bad{ stroke:var(--text); stroke-width:3; fill:none; }
</style>
<defs><marker id="m1c-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="m1c-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="m1c-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker></defs>
<g class="m1c">
<rect x="255" y="96" width="210" height="58" rx="7" class="dark"/>
<text x="360.0" y="120.5" text-anchor="middle" class="on" style="fill:var(--bg)">label switching</text>
<text x="360.0" y="138.5" text-anchor="middle" class="on" style="fill:var(--bg)">one exact-match lookup</text>
<rect x="10" y="16" width="250" height="58" rx="7" class="acc"/>
<text x="135.0" y="40.5" text-anchor="middle" class="b">BGP-free core</text>
<text x="135.0" y="58.5" text-anchor="middle" class="s">P routers carry no customer routes</text>
<rect x="460" y="16" width="250" height="58" rx="7" class="acc"/>
<text x="585.0" y="40.5" text-anchor="middle" class="b">VPNs</text>
<text x="585.0" y="58.5" text-anchor="middle" class="s">many customers, one network</text>
<rect x="10" y="176" width="250" height="58" rx="7" class="acc"/>
<text x="135.0" y="200.5" text-anchor="middle" class="b">Traffic engineering</text>
<text x="135.0" y="218.5" text-anchor="middle" class="s">send traffic off the shortest path</text>
<rect x="460" y="176" width="250" height="58" rx="7" class="acc"/>
<text x="585.0" y="200.5" text-anchor="middle" class="b">Fast reroute</text>
<text x="585.0" y="218.5" text-anchor="middle" class="s">repair a failure in ~50 ms</text>
<line x1="290" y1="96" x2="230" y2="76" class="ag"/>
<line x1="430" y1="96" x2="490" y2="76" class="ag"/>
<line x1="290" y1="154" x2="230" y2="174" class="ag"/>
<line x1="430" y1="154" x2="490" y2="174" class="ag"/>
</g>
</svg>
<figcaption>Why operators still run MPLS: the label is a tunnel, and tunnels make these four services simple.</figcaption>
</figure>

- **A BGP-free core.** Only the edge routers need to know about customer and Internet routes. The core routers carry labels toward the edge routers, so they only need routes to each other. Chapter 5 shows this.
- **VPNs.** Because the core never looks inside, it does not matter that two customers both use `192.168.1.0/24`. The edge routers keep them in separate tables and use a second label to tell them apart. This is the BGP/MPLS IP VPN of Chapters 6 and 7, the most widely sold MPLS service. Layer 2 VPNs (Chapter 8) do the same for Ethernet.
- **Traffic engineering.** IP routing always follows the shortest path. A label can pin traffic to any path you choose, for example to keep a backup link busy or to avoid a congested one. Chapter 11 covers RSVP-TE and SR policies.
- **Fast reroute.** Routers can pre-install a backup tunnel around each link or node, and switch to it within tens of milliseconds when a failure is detected, long before the routing protocol reconverges. Chapter 10 covers this.

## Where MPLS sits in a network

<figure class="diagram">
<svg viewBox="0 0 720 250" style="width:100%;height:auto;max-width:720px" role="img" aria-label="A provider network: customer edge routers CE1 and CE2 sit at customer sites; provider edge routers PE1 and PE2 face customers; provider routers P1 and P2 form the core. Labels are used only between PE1 and PE2." xmlns="http://www.w3.org/2000/svg">
<style>
.m1d{ font-family:var(--sans),sans-serif; }
.m1d .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.m1d .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.m1d .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.m1d .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.m1d .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.m1d .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.m1d .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.m1d .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.m1d .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.m1d .h{ fill:var(--text); font-size:15px; font-weight:700; }
.m1d .b{ fill:var(--text); font-size:13px; font-weight:700; }
.m1d .s{ fill:var(--text-dim); font-size:13px; }
.m1d .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.m1d .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.m1d .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.m1d .c{ fill:var(--text-faint); font-size:12.5px; }
.m1d .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.m1d .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.m1d .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.m1d .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m1d-a); }
.m1d .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m1d-a); marker-start:url(#m1d-s); }
.m1d .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#m1d-g); }
.m1d .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#m1d-a); }
.m1d .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.m1d .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.m1d .bad{ stroke:var(--text); stroke-width:3; fill:none; }
</style>
<defs><marker id="m1d-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="m1d-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="m1d-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker></defs>
<g class="m1d">
<rect x="130" y="30" width="460" height="150" rx="7" class="zone"/>
<text x="360" y="52" text-anchor="middle" class="c">provider network (MPLS)</text>
<circle cx="60" cy="120" r="24" class="pnl"/>
<text x="60" y="124.5" text-anchor="middle" class="b">CE1</text>
<circle cx="660" cy="120" r="24" class="pnl"/>
<text x="660" y="124.5" text-anchor="middle" class="b">CE2</text>
<circle cx="190" cy="120" r="24" class="dark"/>
<text x="190" y="124.5" text-anchor="middle" class="on" style="fill:var(--bg)">PE1</text>
<circle cx="310" cy="120" r="24" class="pnl"/>
<text x="310" y="124.5" text-anchor="middle" class="b">P1</text>
<circle cx="410" cy="120" r="24" class="pnl"/>
<text x="410" y="124.5" text-anchor="middle" class="b">P2</text>
<circle cx="530" cy="120" r="24" class="dark"/>
<text x="530" y="124.5" text-anchor="middle" class="on" style="fill:var(--bg)">PE2</text>
<line x1="84" y1="120" x2="166" y2="120" class="lk"/>
<line x1="214" y1="120" x2="286" y2="120" class="lk"/>
<line x1="334" y1="120" x2="386" y2="120" class="lk"/>
<line x1="434" y1="120" x2="506" y2="120" class="lk"/>
<line x1="554" y1="120" x2="636" y2="120" class="lk"/>
<text x="60" y="165" text-anchor="middle" class="c">customer</text>
<text x="660" y="165" text-anchor="middle" class="c">customer</text>
<text x="190" y="165" text-anchor="middle" class="c">edge: classify</text>
<text x="360" y="165" text-anchor="middle" class="c">core: switch labels</text>
<text x="530" y="165" text-anchor="middle" class="c">edge: deliver</text>
<path d="M190 200 L530 200" class="ar2"/>
<text x="360" y="226" text-anchor="middle" class="s">labelled traffic (the LSP) only inside the provider network</text>
</g>
</svg>
<figcaption>The three roles. CE routers know nothing about MPLS; PE routers do the hard work; P routers only switch labels.</figcaption>
</figure>

Service provider language names three roles:

- **CE (customer edge):** the customer's router. It runs ordinary IP and knows nothing about MPLS.
- **PE (provider edge):** the provider's router facing customers. It is the LER: it classifies traffic, pushes labels, and keeps per-customer state.
- **P (provider):** a core router. It is an LSR that only swaps labels between PEs.

Notice where the complexity lives: almost all of it is in the PEs. P routers can be simple, fast, and stable, which is exactly what you want in the core.

## How labels are agreed

Labels have to be agreed between neighbours: if P1 sends label 37 to P2, P2 must know what 37 means. Different protocols do that job, and much of this book is about them:

| Protocol | What it distributes | Chapter |
|---|---|---|
| Static configuration | Labels typed in by hand | 2 |
| **LDP** | A label for each IGP prefix, hop by hop | 4 |
| **BGP** | Labels for VPN routes and for routes between domains | 5, 6 |
| **RSVP-TE** | Labels along an explicitly chosen path | 11 |
| **IS-IS / OSPF with Segment Routing** | Global labels carried inside the IGP itself | 9 |

The data plane is the same in every case: push, swap, pop.

## Summary

- MPLS forwards on a short **label** chosen at the network edge, instead of on the destination address at every hop.
- **LERs** push and pop labels; **LSRs** swap them. A **LSP** is one-directional.
- The original reason, faster lookups, no longer matters. MPLS survives because a label is a **tunnel**.
- Tunnels enable a **BGP-free core**, **VPNs**, **traffic engineering**, and **fast reroute**.
- **CE**, **PE**, and **P** routers have very different jobs; PEs carry most of the complexity.

The next chapter opens up the label itself and builds an LSP by hand.
