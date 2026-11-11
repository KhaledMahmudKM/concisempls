# Chapter 10: Fast Reroute

When a link or router fails, a link-state IGP recovers on its own: routers detect the failure, flood the news, rerun SPF, and update their forwarding tables. In a well-tuned network this takes a few hundred milliseconds, sometimes more. Voice, video, and many financial applications notice. **Fast reroute** shortens the outage to the time it takes to detect the failure, typically under 50 milliseconds, by installing backup paths before anything fails.

## Convergence versus protection

<figure class="diagram">
<svg viewBox="0 0 720 230" style="width:100%;height:auto;max-width:720px" role="img" aria-label="Convergence after a link failure: without fast reroute, traffic is lost through failure detection, LSP flooding, SPF computation, and forwarding table updates, typically hundreds of milliseconds or more. With fast reroute, the pre-installed backup takes over as soon as the failure is detected, typically within 50 milliseconds." xmlns="http://www.w3.org/2000/svg">
<style>
.m10a{ font-family:var(--sans),sans-serif; }
.m10a .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.m10a .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.m10a .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.m10a .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.m10a .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.m10a .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.m10a .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.m10a .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.m10a .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.m10a .h{ fill:var(--text); font-size:15px; font-weight:700; }
.m10a .b{ fill:var(--text); font-size:13px; font-weight:700; }
.m10a .s{ fill:var(--text-dim); font-size:13px; }
.m10a .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.m10a .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.m10a .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.m10a .c{ fill:var(--text-faint); font-size:12.5px; }
.m10a .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.m10a .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.m10a .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.m10a .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m10a-a); }
.m10a .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m10a-a); marker-start:url(#m10a-s); }
.m10a .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#m10a-g); }
.m10a .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#m10a-a); }
.m10a .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.m10a .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.m10a .bad{ stroke:var(--text); stroke-width:3; fill:none; }
</style>
<defs><marker id="m10a-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="m10a-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="m10a-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker></defs>
<g class="m10a">
<text x="20" y="40" text-anchor="start" class="b">IGP only</text>
<rect x="140" y="22" width="90" height="32" rx="3" class="pnl"/>
<text x="185.0" y="42.5" text-anchor="middle" class="s">detect</text>
<rect x="230" y="22" width="110" height="32" rx="3" class="pnl"/>
<text x="285.0" y="42.5" text-anchor="middle" class="s">flood</text>
<rect x="340" y="22" width="110" height="32" rx="3" class="pnl"/>
<text x="395.0" y="42.5" text-anchor="middle" class="s">SPF</text>
<rect x="450" y="22" width="200" height="32" rx="3" class="pnl"/>
<text x="550.0" y="42.5" text-anchor="middle" class="s">update FIB</text>
<text x="658" y="44" text-anchor="start" class="c">loss</text>
<text x="20" y="112" text-anchor="start" class="b">with fast reroute</text>
<rect x="140" y="94" width="90" height="32" rx="3" class="pnl"/>
<text x="185.0" y="114.5" text-anchor="middle" class="s">detect</text>
<rect x="230" y="94" width="60" height="32" rx="3" class="dark"/>
<text x="260.0" y="114.5" text-anchor="middle" class="on" style="fill:var(--bg)">switch</text>
<text x="298" y="116" text-anchor="start" class="a">backup already in the FIB</text>
<line x1="140" y1="160" x2="700" y2="160" class="ag"/>
<text x="140" y="184" text-anchor="middle" class="c">failure</text>
<text x="290" y="184" text-anchor="middle" class="a">~50 ms</text>
<text x="650" y="184" text-anchor="middle" class="c">100s of ms to seconds</text>
<text x="360" y="214" text-anchor="middle" class="c">the IGP still reconverges afterwards; fast reroute only covers the gap</text>
</g>
</svg>
<figcaption>Fast reroute does not replace convergence. It hides it, by switching to a precomputed backup the moment a failure is seen.</figcaption>
</figure>

The IGP's recovery has four phases: **detect** the failure (instantly for a dead interface, or within tens of milliseconds with BFD), **flood** the change, run **SPF**, and **update** the forwarding table, which can take a long time when many prefixes are affected.

Fast reroute is local. The router next to the failure, the **point of local repair (PLR)**, has already computed and installed a backup next hop for each destination. When it detects the failure, it switches to the backup immediately, without waiting for anyone else. Traffic flows on the backup for the next few hundred milliseconds while the IGP converges normally, after which every router uses its new best path and the backup is no longer needed.

## Loop-free alternates

The simplest backup is a different neighbour, provided that neighbour will not send the traffic straight back.

<figure class="diagram">
<svg viewBox="0 0 720 270" style="width:100%;height:auto;max-width:720px" role="img" aria-label="A loop-free alternate: router S normally sends traffic for D to E. Its neighbour N is a loop-free alternate if N&#x27;s own shortest path to D does not come back through S, which holds when distance(N, D) is less than distance(N, S) plus distance(S, D)." xmlns="http://www.w3.org/2000/svg">
<style>
.m10b{ font-family:var(--sans),sans-serif; }
.m10b .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.m10b .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.m10b .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.m10b .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.m10b .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.m10b .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.m10b .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.m10b .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.m10b .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.m10b .h{ fill:var(--text); font-size:15px; font-weight:700; }
.m10b .b{ fill:var(--text); font-size:13px; font-weight:700; }
.m10b .s{ fill:var(--text-dim); font-size:13px; }
.m10b .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.m10b .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.m10b .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.m10b .c{ fill:var(--text-faint); font-size:12.5px; }
.m10b .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.m10b .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.m10b .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.m10b .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m10b-a); }
.m10b .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m10b-a); marker-start:url(#m10b-s); }
.m10b .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#m10b-g); }
.m10b .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#m10b-a); }
.m10b .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.m10b .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.m10b .bad{ stroke:var(--text); stroke-width:3; fill:none; }
</style>
<defs><marker id="m10b-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="m10b-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="m10b-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker></defs>
<g class="m10b">
<circle cx="120" cy="80" r="24" class="dark"/>
<text x="120" y="84.5" text-anchor="middle" class="on" style="fill:var(--bg)">S</text>
<circle cx="400" cy="80" r="24" class="pnl"/>
<text x="400" y="84.5" text-anchor="middle" class="b">E</text>
<circle cx="620" cy="150" r="24" class="acc"/>
<text x="620" y="154.5" text-anchor="middle" class="b">D</text>
<circle cx="260" cy="200" r="24" class="pnl"/>
<text x="260" y="204.5" text-anchor="middle" class="b">N</text>
<line x1="144" y1="80" x2="376" y2="80" class="out"/>
<line x1="424" y1="86" x2="597" y2="143" class="out"/>
<line x1="138" y1="96" x2="242" y2="184" class="lk"/>
<line x1="284" y1="196" x2="596" y2="156" class="lk"/>
<line x1="250" y1="72" x2="274" y2="96" class="bad"/>
<line x1="274" y1="72" x2="250" y2="96" class="bad"/>
<text x="260" y="64" text-anchor="middle" class="c">failure</text>
<text x="470" y="196" text-anchor="middle" class="c">N's own path to D</text>
<text x="330" y="70" text-anchor="middle" class="a">primary</text>
<rect x="20" y="228" width="680" height="36" rx="7" class="acc"/>
<text x="360.0" y="250.5" text-anchor="middle" class="m">loop-free if  dist(N, D)  &lt;  dist(N, S) + dist(S, D)</text>
</g>
</svg>
<figcaption>If the neighbour would not send the traffic straight back, S can hand it over the moment its link to E fails.</figcaption>
</figure>

A neighbour N of router S is a **loop-free alternate (LFA)** for destination D if N's own shortest path to D does not pass through S. In metric terms:

> dist(N, D) < dist(N, S) + dist(S, D)

If that holds, S can hand traffic for D to N the moment its primary link fails, and N will forward it correctly using its normal routes. LFA needs no tunnels and no new protocols, only a little extra SPF computation on the PLR (RFC 5286).

LFA's weakness is coverage. Whether an alternate exists depends entirely on the topology and metrics.

<figure class="diagram">
<svg viewBox="0 0 720 260" style="width:100%;height:auto;max-width:720px" role="img" aria-label="A ring of six routers with equal metrics where no LFA exists: S&#x27;s only other neighbour N reaches D by sending traffic back through S, so handing traffic to N would create a loop." xmlns="http://www.w3.org/2000/svg">
<style>
.m10c{ font-family:var(--sans),sans-serif; }
.m10c .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.m10c .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.m10c .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.m10c .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.m10c .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.m10c .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.m10c .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.m10c .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.m10c .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.m10c .h{ fill:var(--text); font-size:15px; font-weight:700; }
.m10c .b{ fill:var(--text); font-size:13px; font-weight:700; }
.m10c .s{ fill:var(--text-dim); font-size:13px; }
.m10c .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.m10c .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.m10c .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.m10c .c{ fill:var(--text-faint); font-size:12.5px; }
.m10c .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.m10c .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.m10c .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.m10c .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m10c-a); }
.m10c .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m10c-a); marker-start:url(#m10c-s); }
.m10c .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#m10c-g); }
.m10c .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#m10c-a); }
.m10c .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.m10c .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.m10c .bad{ stroke:var(--text); stroke-width:3; fill:none; }
</style>
<defs><marker id="m10c-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="m10c-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="m10c-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker></defs>
<g class="m10c">
<line x1="226.5" y1="110.5" x2="268.5" y2="65.8" class="lk"/>
<line x1="309.0" y1="48.3" x2="411.0" y2="48.3" class="lk"/>
<line x1="451.5" y1="65.8" x2="493.5" y2="110.5" class="lk"/>
<line x1="493.5" y1="145.5" x2="451.5" y2="190.2" class="lk"/>
<line x1="411.0" y1="207.7" x2="309.0" y2="207.7" class="lk"/>
<line x1="268.5" y1="190.2" x2="226.5" y2="145.5" class="lk"/>
<circle cx="210.0" cy="128.0" r="24" class="dark"/>
<text x="210.0" y="132.5" text-anchor="middle" class="on" style="fill:var(--bg)">S</text>
<circle cx="285.0" cy="48.3" r="24" class="pnl"/>
<text x="285.0" y="52.8" text-anchor="middle" class="b">E</text>
<circle cx="435.0" cy="48.3" r="24" class="acc"/>
<text x="435.0" y="52.8" text-anchor="middle" class="b">D</text>
<circle cx="510.0" cy="128.0" r="24" class="pnl"/>
<text x="510.0" y="132.5" text-anchor="middle" class="b">R3</text>
<circle cx="435.0" cy="207.7" r="24" class="pnl"/>
<text x="435.0" y="212.2" text-anchor="middle" class="b">R2</text>
<circle cx="285.0" cy="207.7" r="24" class="pnl"/>
<text x="285.0" y="212.2" text-anchor="middle" class="b">N</text>
<text x="130" y="60" text-anchor="middle" class="c">S to E fails</text>
<text x="130" y="210" text-anchor="middle" class="c">N's shortest path to D</text>
<text x="130" y="226" text-anchor="middle" class="c">is back through S</text>
</g>
</svg>
<figcaption>Plain LFA leaves gaps in common topologies such as rings. Remote LFA and TI-LFA close them with tunnels.</figcaption>
</figure>

In a ring, a classic case, the other neighbour reaches D by going back through S, so no LFA exists for many destinations. Real networks often have LFA coverage between 60 and 90 percent, leaving holes exactly where operators least expect them.

## Remote LFA

If no direct neighbour is safe, perhaps a router further away is. **Remote LFA** (RFC 7490) tunnels traffic to such a router.

<figure class="diagram">
<svg viewBox="0 0 720 270" style="width:100%;height:auto;max-width:720px" role="img" aria-label="Remote LFA: the P space of S is the set of routers S can reach without using the failed link; the Q space of E is the set of routers that reach E without using it. A router in both, the PQ node, is a safe tunnel endpoint; S tunnels traffic to it with LDP, and from there the traffic proceeds normally." xmlns="http://www.w3.org/2000/svg">
<style>
.m10d{ font-family:var(--sans),sans-serif; }
.m10d .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.m10d .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.m10d .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.m10d .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.m10d .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.m10d .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.m10d .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.m10d .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.m10d .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.m10d .h{ fill:var(--text); font-size:15px; font-weight:700; }
.m10d .b{ fill:var(--text); font-size:13px; font-weight:700; }
.m10d .s{ fill:var(--text-dim); font-size:13px; }
.m10d .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.m10d .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.m10d .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.m10d .c{ fill:var(--text-faint); font-size:12.5px; }
.m10d .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.m10d .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.m10d .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.m10d .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m10d-a); }
.m10d .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m10d-a); marker-start:url(#m10d-s); }
.m10d .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#m10d-g); }
.m10d .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#m10d-a); }
.m10d .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.m10d .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.m10d .bad{ stroke:var(--text); stroke-width:3; fill:none; }
</style>
<defs><marker id="m10d-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="m10d-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="m10d-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker></defs>
<g class="m10d">
<rect x="20" y="30" width="300" height="170" rx="7" class="tint"/>
<text x="170" y="54" text-anchor="middle" class="a">P space of S</text>
<rect x="400" y="30" width="300" height="170" rx="7" class="tint"/>
<text x="550" y="54" text-anchor="middle" class="a">Q space of E</text>
<rect x="250" y="70" width="220" height="100" rx="7" class="dash"/>
<circle cx="80" cy="130" r="24" class="dark"/>
<text x="80" y="134.5" text-anchor="middle" class="on" style="fill:var(--bg)">S</text>
<circle cx="640" cy="130" r="24" class="pnl"/>
<text x="640" y="134.5" text-anchor="middle" class="b">E</text>
<circle cx="360" cy="120" r="28" class="acc"/>
<text x="360" y="124.5" text-anchor="middle" class="b">PQ</text>
<path d="M104 130 C 180 170, 280 160, 332 128" class="ar"/>
<text x="190" y="186" text-anchor="middle" class="c">tunnel to the PQ node</text>
<path d="M388 120 C 460 100, 560 110, 616 126" class="ag"/>
<text x="520" y="186" text-anchor="middle" class="c">normal forwarding from there</text>
<text x="360" y="236" text-anchor="middle" class="c">remote LFA needs a targeted LDP session to the PQ node, and still misses some cases</text>
</g>
</svg>
<figcaption>Remote LFA extends coverage with a tunnel, but not to every topology.</figcaption>
</figure>

- The **P space** of S contains the routers S can reach without using the protected link.
- The **Q space** of E contains the routers that can reach E without using it.
- A **PQ node**, in both, is a safe tunnel endpoint: S can get traffic there, and from there it reaches its destination normally.

S tunnels the traffic to the PQ node with an LDP label, which requires a **targeted LDP** session to learn the PQ node's label for the destination. Remote LFA closes many of LFA's gaps but not all, and it still sends traffic along paths that may differ from where the network settles after convergence.

## TI-LFA

**Topology-Independent LFA (TI-LFA)** uses Segment Routing to remove the topology dependence altogether. For each destination, the PLR computes the **post-convergence path**: the path traffic will take once the IGP has reconverged without the failed link or node. It then encodes that path as a short **repair segment list** and installs it as the backup.

<figure class="diagram">
<svg viewBox="0 0 720 290" style="width:100%;height:auto;max-width:720px" role="img" aria-label="TI-LFA in the lab core: p1 forwards traffic for pe2 over the link to p2. TI-LFA computes the path traffic would take after the link failed, through p3, and pre-installs it as a repair segment list, here simply the node SID of pe2 sent toward p3. When the link fails, p1 switches at once." xmlns="http://www.w3.org/2000/svg">
<style>
.m10e{ font-family:var(--sans),sans-serif; }
.m10e .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.m10e .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.m10e .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.m10e .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.m10e .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.m10e .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.m10e .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.m10e .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.m10e .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.m10e .h{ fill:var(--text); font-size:15px; font-weight:700; }
.m10e .b{ fill:var(--text); font-size:13px; font-weight:700; }
.m10e .s{ fill:var(--text-dim); font-size:13px; }
.m10e .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.m10e .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.m10e .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.m10e .c{ fill:var(--text-faint); font-size:12.5px; }
.m10e .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.m10e .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.m10e .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.m10e .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m10e-a); }
.m10e .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m10e-a); marker-start:url(#m10e-s); }
.m10e .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#m10e-g); }
.m10e .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#m10e-a); }
.m10e .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.m10e .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.m10e .bad{ stroke:var(--text); stroke-width:3; fill:none; }
</style>
<defs><marker id="m10e-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="m10e-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="m10e-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker></defs>
<g class="m10e">
<line x1="114.0" y1="80.0" x2="246.0" y2="80.0" class="lk"/>
<line x1="294.0" y1="80.0" x2="426.0" y2="80.0" class="out"/>
<line x1="474.0" y1="80.0" x2="606.0" y2="80.0" class="out"/>
<line x1="111.6" y1="90.4" x2="338.4" y2="199.6" class="lk"/>
<line x1="381.6" y1="199.6" x2="608.4" y2="90.4" class="lk"/>
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
<line x1="345" y1="66" x2="375" y2="94" class="bad"/>
<line x1="375" y1="66" x2="345" y2="94" class="bad"/>
<path d="M284 110 C 300 160, 320 180, 336 196" class="ar"/>
<path d="M384 196 C 470 160, 560 130, 606 100" class="ar"/>
<rect x="400" y="210" width="310" height="60" rx="7" class="acc"/>
<text x="555.0" y="235.5" text-anchor="middle" class="m">repair on p1: push 16002, out to p3</text>
<text x="555.0" y="253.5" text-anchor="middle" class="s">the post-convergence path</text>
<text x="450" y="54" text-anchor="middle" class="c">primary: p1, p2, pe2</text>
</g>
</svg>
<figcaption>TI-LFA protects along the path the network will use after convergence, so there is no second traffic shift.</figcaption>
</figure>

Because any path can be written as a segment list (a node SID to reach the point where the new path diverges, perhaps an adjacency SID to cross a specific link), TI-LFA protects every destination against any single link failure in any topology, and against node failures where the topology allows any protection at all. Because the repair path is the post-convergence path, traffic moves only once: from the primary to the repair path, which is already where the IGP will send it. Most repair lists are one or two labels long.

TI-LFA can protect against:

- **link failure** (the default);
- **node failure**, avoiding the next router entirely;
- **SRLG failure**, avoiding all links in a shared risk link group, such as fibres in the same duct.

### Microloops

<figure class="diagram">
<svg viewBox="0 0 720 220" style="width:100%;height:auto;max-width:720px" role="img" aria-label="A microloop: after a failure, routers update their forwarding tables at slightly different times. For a moment router A already sends traffic to B while B still sends it back to A, and packets loop until both have converged." xmlns="http://www.w3.org/2000/svg">
<style>
.m10f{ font-family:var(--sans),sans-serif; }
.m10f .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.m10f .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.m10f .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.m10f .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.m10f .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.m10f .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.m10f .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.m10f .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.m10f .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.m10f .h{ fill:var(--text); font-size:15px; font-weight:700; }
.m10f .b{ fill:var(--text); font-size:13px; font-weight:700; }
.m10f .s{ fill:var(--text-dim); font-size:13px; }
.m10f .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.m10f .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.m10f .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.m10f .c{ fill:var(--text-faint); font-size:12.5px; }
.m10f .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.m10f .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.m10f .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.m10f .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m10f-a); }
.m10f .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m10f-a); marker-start:url(#m10f-s); }
.m10f .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#m10f-g); }
.m10f .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#m10f-a); }
.m10f .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.m10f .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.m10f .bad{ stroke:var(--text); stroke-width:3; fill:none; }
</style>
<defs><marker id="m10f-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="m10f-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="m10f-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker></defs>
<g class="m10f">
<circle cx="200" cy="90" r="24" class="pnl"/>
<text x="200" y="94.5" text-anchor="middle" class="b">A</text>
<circle cx="520" cy="90" r="24" class="pnl"/>
<text x="520" y="94.5" text-anchor="middle" class="b">B</text>
<path d="M224 80 L496 80" class="ar"/>
<path d="M496 102 L224 102" class="ar"/>
<text x="360" y="66" text-anchor="middle" class="c">A has converged: send via B</text>
<text x="360" y="130" text-anchor="middle" class="c">B has not yet: send via A</text>
<rect x="120" y="160" width="480" height="40" rx="7" class="pnl"/>
<text x="360.0" y="184.5" text-anchor="middle" class="s">SR can prevent this by steering briefly along the new path</text>
</g>
</svg>
<figcaption>Microloops are short, but they drop traffic. TI-LFA and SR microloop avoidance address them together.</figcaption>
</figure>

Even with protection in place, the moment the IGP reconverges can cause trouble. Routers update their tables at slightly different times, and two neighbours can briefly point at each other, a **microloop**. With SR, a router can temporarily steer traffic along the new path with a segment list until all its neighbours have converged, which prevents microloops; this is called **microloop avoidance**.

## Configuring TI-LFA in FRR

TI-LFA in FRR is enabled per interface in IS-IS, on every core interface of every router:

```text
interface eth1
 ip router isis CORE
 isis network point-to-point
 isis fast-reroute ti-lfa
exit
```

Add `node-protection` to protect against the failure of the next router, and `link-fallback` to fall back to link protection where node protection is impossible. Plain LFA and remote LFA are available too, with `isis fast-reroute lfa` and `isis fast-reroute remote-lfa tunnel mpls-ldp`.

## Hands on: watching the backup

The SR lab from Chapter 9 (`labs/lab5-sr-mpls`) already has TI-LFA on every core interface. Focus on p1, which forwards traffic from pe1 to pe2 over its link to p2.

```bash
cd labs/lab5-sr-mpls
sudo containerlab deploy -t lab5-sr-mpls.clab.yml     # if not already running
docker exec -it clab-lab5-sr-mpls-p1 vtysh
```

```text
p1# show isis fast-reroute summary        ! how many prefixes are protected
p1# show isis route backup                ! the repair paths per destination
p1# show ip route 10.0.0.2/32             ! primary next hop plus a backup
```

The route to pe2's loopback shows the primary next hop, p2, and a backup through p3 with its label stack. In this small topology the repair is simple, because p3's own shortest path to pe2 does not come back through p1; the backup is effectively an LFA expressed with SR labels. On larger or ring-shaped topologies the same command shows repair lists with two or more labels.

Now fail the protected link while traffic flows. The Alpine hosts use BusyBox `ping`; for fast probes, install the full `ping` first (this needs Internet access from the lab host). In one terminal, run a fast ping from ce1 to ce2; in another, take the link down on p1:

```bash
docker exec clab-lab5-sr-mpls-ce1 apk add iputils
docker exec -it clab-lab5-sr-mpls-ce1 ping -i 0.01 -I 192.168.1.1 192.168.2.1
sudo ip netns exec clab-lab5-sr-mpls-p1 ip link set eth2 down
```

Stop the ping with Ctrl+C and compare transmitted and received packets; each lost reply is 10 ms of outage. Repeat the experiment after removing `isis fast-reroute ti-lfa` from p1's interfaces, and compare. Container labs are not precise timing instruments, so treat the numbers as an illustration, not a benchmark. Bring the link back with `ip link set eth2 up`.

## Summary

- **Fast reroute** pre-installs backups so the **PLR** can switch within tens of milliseconds, while the IGP converges normally.
- An **LFA** neighbour must not route back through the PLR: dist(N, D) < dist(N, S) + dist(S, D). Coverage depends on topology.
- **Remote LFA** tunnels to a **PQ node** with targeted LDP; better coverage, still incomplete.
- **TI-LFA** encodes the **post-convergence path** as a segment list, protecting against any single **link**, **node**, or **SRLG** failure.
- SR also enables **microloop avoidance**. In FRR, enable TI-LFA with `isis fast-reroute ti-lfa` on each core interface.
