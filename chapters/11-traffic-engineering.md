# Chapter 11: Traffic Engineering

IP routing sends all traffic to a destination along the single best path, or a few equal ones. When that path fills up, nothing in the routing protocol notices, and capacity elsewhere goes unused. **Traffic engineering (TE)** is the practice of placing traffic on paths deliberately, according to bandwidth, latency, or policy. MPLS made TE practical, first with **RSVP-TE** tunnels and now with **Segment Routing policies**.

## The problem

<figure class="diagram">
<svg viewBox="0 0 720 260" style="width:100%;height:auto;max-width:720px" role="img" aria-label="The fish problem: traffic from A and from B joins at C, and all of it follows the IGP shortest path through D to F, congesting it, while the slightly longer path through E sits idle. Changing metrics just moves the congestion." xmlns="http://www.w3.org/2000/svg">
<style>
.m11a{ font-family:var(--sans),sans-serif; }
.m11a .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.m11a .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.m11a .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.m11a .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.m11a .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.m11a .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.m11a .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.m11a .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.m11a .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.m11a .h{ fill:var(--text); font-size:15px; font-weight:700; }
.m11a .b{ fill:var(--text); font-size:13px; font-weight:700; }
.m11a .s{ fill:var(--text-dim); font-size:13px; }
.m11a .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.m11a .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.m11a .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.m11a .c{ fill:var(--text-faint); font-size:12.5px; }
.m11a .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.m11a .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.m11a .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.m11a .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m11a-a); }
.m11a .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m11a-a); marker-start:url(#m11a-s); }
.m11a .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#m11a-g); }
.m11a .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#m11a-a); }
.m11a .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.m11a .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.m11a .bad{ stroke:var(--text); stroke-width:3; fill:none; }
</style>
<defs><marker id="m11a-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="m11a-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="m11a-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker></defs>
<g class="m11a">
<circle cx="60" cy="60" r="24" class="pnl"/>
<text x="60" y="64.5" text-anchor="middle" class="b">A</text>
<circle cx="60" cy="190" r="24" class="pnl"/>
<text x="60" y="194.5" text-anchor="middle" class="b">B</text>
<circle cx="200" cy="125" r="24" class="dark"/>
<text x="200" y="129.5" text-anchor="middle" class="on" style="fill:var(--bg)">C</text>
<circle cx="400" cy="60" r="24" class="pnl"/>
<text x="400" y="64.5" text-anchor="middle" class="b">D</text>
<circle cx="400" cy="190" r="24" class="pnl"/>
<text x="400" y="194.5" text-anchor="middle" class="b">E</text>
<circle cx="620" cy="125" r="24" class="dark"/>
<text x="620" y="129.5" text-anchor="middle" class="on" style="fill:var(--bg)">F</text>
<line x1="80" y1="70" x2="180" y2="115" class="lk"/>
<line x1="80" y1="180" x2="180" y2="135" class="lk"/>
<line x1="222" y1="113" x2="378" y2="70" class="out"/>
<line x1="422" y1="70" x2="598" y2="115" class="out"/>
<line x1="222" y1="137" x2="378" y2="180" class="lk"/>
<line x1="422" y1="180" x2="598" y2="135" class="lk"/>
<text x="300" y="66" text-anchor="middle" class="a">congested</text>
<text x="520" y="66" text-anchor="middle" class="c">shortest path</text>
<text x="300" y="196" text-anchor="middle" class="c">idle</text>
<text x="520" y="196" text-anchor="middle" class="c">cost +10</text>
<text x="360" y="246" text-anchor="middle" class="c">an IGP sends everything over the single best path; TE spreads it deliberately</text>
</g>
</svg>
<figcaption>The classic motivation for traffic engineering: capacity exists, but shortest-path routing will not use it.</figcaption>
</figure>

In the classic "fish" picture, traffic from A and B merges at C, and the IGP sends all of it through D, even though the path through E is only slightly longer. Tweaking metrics only moves the whole load from one path to the other. What is needed is a way to send **some** traffic one way and the rest another, which destination-based routing cannot express but a label can.

## RSVP-TE

> **No lab.** FRR has no RSVP-TE implementation, and Linux cannot act as an RSVP-TE router. This section uses diagrams only; the SR policy lab later in the chapter shows the same idea in practice.

**RSVP-TE** (RFC 3209) builds an LSP along an explicitly chosen path and reserves bandwidth on it. The head end computes the path, then signals it.

### Computing the path: CSPF

The IGP floods extra information for TE: each link's maximum bandwidth, how much is already reserved, administrative groups ("colours" of links), and TE metrics. Every head end builds a **TE database** from it, and computes paths with **constrained SPF (CSPF)**.

<figure class="diagram">
<svg viewBox="0 0 720 270" style="width:100%;height:auto;max-width:720px" role="img" aria-label="Constrained shortest path first: to place a 300 megabit tunnel, the head end first removes every link with less than 300 megabits of unreserved bandwidth, then runs a shortest path computation on what remains." xmlns="http://www.w3.org/2000/svg">
<style>
.m11c{ font-family:var(--sans),sans-serif; }
.m11c .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.m11c .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.m11c .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.m11c .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.m11c .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.m11c .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.m11c .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.m11c .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.m11c .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.m11c .h{ fill:var(--text); font-size:15px; font-weight:700; }
.m11c .b{ fill:var(--text); font-size:13px; font-weight:700; }
.m11c .s{ fill:var(--text-dim); font-size:13px; }
.m11c .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.m11c .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.m11c .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.m11c .c{ fill:var(--text-faint); font-size:12.5px; }
.m11c .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.m11c .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.m11c .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.m11c .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m11c-a); }
.m11c .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m11c-a); marker-start:url(#m11c-s); }
.m11c .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#m11c-g); }
.m11c .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#m11c-a); }
.m11c .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.m11c .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.m11c .bad{ stroke:var(--text); stroke-width:3; fill:none; }
</style>
<defs><marker id="m11c-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="m11c-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="m11c-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker></defs>
<g class="m11c">
<line x1="114.0" y1="80.0" x2="246.0" y2="80.0" class="lk"/>
<line x1="294.0" y1="80.0" x2="426.0" y2="80.0" class="lk"/>
<line x1="474.0" y1="80.0" x2="606.0" y2="80.0" class="lk"/>
<line x1="111.6" y1="90.4" x2="338.4" y2="199.6" class="lk"/>
<line x1="381.6" y1="199.6" x2="608.4" y2="90.4" class="lk"/>
<line x1="283.7" y1="99.7" x2="346.3" y2="190.3" class="lk"/>
<circle cx="90" cy="80" r="24" class="dark"/>
<text x="90" y="84.5" text-anchor="middle" class="on" style="fill:var(--bg)">head</text>
<circle cx="270" cy="80" r="24" class="pnl"/>
<text x="270" y="84.5" text-anchor="middle" class="b">p1</text>
<circle cx="450" cy="80" r="24" class="pnl"/>
<text x="450" y="84.5" text-anchor="middle" class="b">p2</text>
<circle cx="630" cy="80" r="24" class="dark"/>
<text x="630" y="84.5" text-anchor="middle" class="on" style="fill:var(--bg)">tail</text>
<circle cx="360" cy="210" r="24" class="pnl"/>
<text x="360" y="214.5" text-anchor="middle" class="b">p3</text>
<rect x="150" y="92" width="60" height="22" rx="4" class="pnl"/>
<text x="180.0" y="107.5" text-anchor="middle" class="s">1G</text>
<rect x="330" y="92" width="70" height="22" rx="4" class="box"/>
<text x="365.0" y="107.5" text-anchor="middle" class="s">200M</text>
<rect x="510" y="92" width="60" height="22" rx="4" class="pnl"/>
<text x="540.0" y="107.5" text-anchor="middle" class="s">1G</text>
<rect x="150" y="160" width="60" height="22" rx="4" class="pnl"/>
<text x="180.0" y="175.5" text-anchor="middle" class="s">1G</text>
<rect x="316" y="128" width="60" height="22" rx="4" class="pnl"/>
<text x="346.0" y="143.5" text-anchor="middle" class="s">1G</text>
<rect x="520" y="170" width="60" height="22" rx="4" class="pnl"/>
<text x="550.0" y="185.5" text-anchor="middle" class="s">1G</text>
<line x1="345" y1="70" x2="375" y2="92" class="bad"/>
<line x1="375" y1="70" x2="345" y2="92" class="bad"/>
<text x="360" y="254" text-anchor="middle" class="c">p1 to p2 has only 200M free: pruned; best remaining path is head, p3, tail</text>
</g>
</svg>
<figcaption>CSPF is plain SPF on a pruned graph. The constraints come from the TE database flooded by the IGP.</figcaption>
</figure>

CSPF is ordinary SPF on a pruned graph: first remove every link that fails the constraints (not enough free bandwidth, wrong link colour), then find the shortest path through what remains.

### Signalling the path

<figure class="diagram">
<svg viewBox="0 0 720 280" style="width:100%;height:auto;max-width:720px" role="img" aria-label="RSVP-TE signalling: the head end sends a PATH message along the explicit route, with the requested bandwidth, hop by hop to the tail end; the tail end replies with a RESV message travelling back along the same path, and each router allocates a label and reserves the bandwidth." xmlns="http://www.w3.org/2000/svg">
<style>
.m11b{ font-family:var(--sans),sans-serif; }
.m11b .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.m11b .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.m11b .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.m11b .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.m11b .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.m11b .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.m11b .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.m11b .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.m11b .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.m11b .h{ fill:var(--text); font-size:15px; font-weight:700; }
.m11b .b{ fill:var(--text); font-size:13px; font-weight:700; }
.m11b .s{ fill:var(--text-dim); font-size:13px; }
.m11b .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.m11b .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.m11b .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.m11b .c{ fill:var(--text-faint); font-size:12.5px; }
.m11b .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.m11b .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.m11b .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.m11b .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m11b-a); }
.m11b .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m11b-a); marker-start:url(#m11b-s); }
.m11b .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#m11b-g); }
.m11b .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#m11b-a); }
.m11b .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.m11b .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.m11b .bad{ stroke:var(--text); stroke-width:3; fill:none; }
</style>
<defs><marker id="m11b-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="m11b-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="m11b-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker></defs>
<g class="m11b">
<circle cx="90" cy="60" r="24" class="dark"/>
<text x="90" y="64.5" text-anchor="middle" class="on" style="fill:var(--bg)">head</text>
<circle cx="270" cy="60" r="24" class="pnl"/>
<text x="270" y="64.5" text-anchor="middle" class="b">R2</text>
<circle cx="450" cy="60" r="24" class="pnl"/>
<text x="450" y="64.5" text-anchor="middle" class="b">R3</text>
<circle cx="630" cy="60" r="24" class="dark"/>
<text x="630" y="64.5" text-anchor="middle" class="on" style="fill:var(--bg)">tail</text>
<line x1="114" y1="60" x2="246" y2="60" class="lk"/>
<line x1="294" y1="60" x2="426" y2="60" class="lk"/>
<line x1="474" y1="60" x2="606" y2="60" class="lk"/>
<path d="M114 104 L606 104" class="ar"/>
<text x="360" y="96" text-anchor="middle" class="a">PATH: explicit route R2, R3, tail; 300 Mb/s</text>
<path d="M606 150 L114 150" class="ad"/>
<text x="360" y="142" text-anchor="middle" class="a">RESV: labels, reservation confirmed</text>
<rect x="210" y="170" width="120" height="30" rx="5" class="acc"/>
<text x="270.0" y="189.5" text-anchor="middle" class="b">label 41</text>
<rect x="390" y="170" width="120" height="30" rx="5" class="acc"/>
<text x="450.0" y="189.5" text-anchor="middle" class="b">label 57</text>
<rect x="570" y="170" width="120" height="30" rx="5" class="acc"/>
<text x="630.0" y="189.5" text-anchor="middle" class="b">implicit null</text>
<text x="360" y="236" text-anchor="middle" class="c">every router on the path keeps state for every tunnel through it</text>
<text x="360" y="258" text-anchor="middle" class="c">and refreshes it periodically</text>
</g>
</svg>
<figcaption>RSVP-TE reserves bandwidth hop by hop. It works, but the per-tunnel state in the core is its scaling limit.</figcaption>
</figure>

The head end sends a **PATH** message hop by hop along the computed route, carried in an **Explicit Route Object (ERO)**, with the requested bandwidth. The tail end answers with a **RESV** message that travels back along the same path; each router allocates a label for the tunnel, reserves the bandwidth, and passes its label upstream. When RESV reaches the head end, the tunnel is up, and its label stack works like any other LSP.

RSVP-TE adds powerful features: bandwidth guarantees, preemption between tunnels of different priority, and **RSVP-TE fast reroute** with one-to-one detours or facility backup tunnels (RFC 4090), the original 50 millisecond protection. Its drawback is state. Every router on the path keeps state for every tunnel through it and refreshes it periodically. A full mesh of tunnels between hundreds of PEs means tens of thousands of tunnels, all of it state in the core. That is the main reason operators are moving to SR.

## Segment Routing policies

With Segment Routing, an explicit path is just a label stack pushed at the head end. Nothing needs signalling, and the core keeps no per-path state. SR packages this as an **SR policy** (RFC 9256).

<figure class="diagram">
<svg viewBox="0 0 720 290" style="width:100%;height:auto;max-width:720px" role="img" aria-label="An SR policy on pe1, identified by color 10 and endpoint 10.0.0.2, with two candidate paths: a dynamic path with preference 200 and an explicit segment list with preference 100, here [16013, 16002] through p3. The valid path with the highest preference is active. The policy also has a binding SID, 15010." xmlns="http://www.w3.org/2000/svg">
<style>
.m11d{ font-family:var(--sans),sans-serif; }
.m11d .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.m11d .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.m11d .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.m11d .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.m11d .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.m11d .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.m11d .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.m11d .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.m11d .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.m11d .h{ fill:var(--text); font-size:15px; font-weight:700; }
.m11d .b{ fill:var(--text); font-size:13px; font-weight:700; }
.m11d .s{ fill:var(--text-dim); font-size:13px; }
.m11d .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.m11d .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.m11d .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.m11d .c{ fill:var(--text-faint); font-size:12.5px; }
.m11d .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.m11d .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.m11d .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.m11d .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m11d-a); }
.m11d .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m11d-a); marker-start:url(#m11d-s); }
.m11d .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#m11d-g); }
.m11d .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#m11d-a); }
.m11d .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.m11d .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.m11d .bad{ stroke:var(--text); stroke-width:3; fill:none; }
</style>
<defs><marker id="m11d-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="m11d-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="m11d-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker></defs>
<g class="m11d">
<rect x="20" y="20" width="300" height="70" rx="7" class="dark"/>
<text x="170.0" y="41.5" text-anchor="middle" class="on" style="fill:var(--bg)">SR policy (headend pe1)</text>
<text x="170.0" y="59.5" text-anchor="middle" class="on" style="fill:var(--bg)">color 10, endpoint 10.0.0.2</text>
<text x="170.0" y="77.5" text-anchor="middle" class="on" style="fill:var(--bg)">binding SID 15010</text>
<rect x="400" y="20" width="300" height="56" rx="7" class="pnl"/>
<text x="550.0" y="43.5" text-anchor="middle" class="b">preference 200: dynamic</text>
<text x="550.0" y="61.5" text-anchor="middle" class="c">computed by the head end or a PCE</text>
<rect x="400" y="96" width="300" height="56" rx="7" class="acc"/>
<text x="550.0" y="119.5" text-anchor="middle" class="b">preference 100: explicit</text>
<text x="550.0" y="137.5" text-anchor="middle" class="m">segment list [16013, 16002]</text>
<line x1="320" y1="48" x2="398" y2="48" class="ag"/>
<line x1="320" y1="70" x2="398" y2="120" class="ar"/>
<text x="550" y="176" text-anchor="middle" class="c">active if the higher one is invalid</text>
<rect x="20" y="210" width="680" height="60" rx="7" class="pnl"/>
<text x="360.0" y="234.0" text-anchor="middle" class="s">An SR policy is identified by (color, endpoint). The color is an intent:</text>
<text x="360.0" y="255.0" text-anchor="middle" class="s">for example 10 = avoid p1-p2, 20 = low latency.</text>
</g>
</svg>
<figcaption>SR policies replace TE tunnels. All state lives at the head end; the core only knows node and adjacency SIDs.</figcaption>
</figure>

- A policy is identified by its **head end**, a **color**, and an **endpoint**. The color is a number that stands for an intent, such as "low latency" or "avoid this link"; the endpoint is the destination router.
- A policy has one or more **candidate paths**, each with a **preference**. A candidate path is either **explicit** (a configured segment list) or **dynamic** (computed by the head end's CSPF or by a controller). The valid candidate with the highest preference is active; if it fails, the next one takes over.
- A policy can have a **binding SID**, a label that stands for the whole policy, so another router can steer traffic into it by pushing a single label.

### Steering traffic into a policy

<figure class="diagram">
<svg viewBox="0 0 720 250" style="width:100%;height:auto;max-width:720px" role="img" aria-label="Steering traffic into an SR policy: a BGP route for 192.168.99.0/24 arrives with next hop 10.0.0.2 and color 10. The head end finds the policy with color 10 and endpoint 10.0.0.2 and sends the route&#x27;s traffic with that policy&#x27;s segment list instead of the shortest path." xmlns="http://www.w3.org/2000/svg">
<style>
.m11e{ font-family:var(--sans),sans-serif; }
.m11e .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.m11e .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.m11e .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.m11e .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.m11e .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.m11e .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.m11e .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.m11e .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.m11e .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.m11e .h{ fill:var(--text); font-size:15px; font-weight:700; }
.m11e .b{ fill:var(--text); font-size:13px; font-weight:700; }
.m11e .s{ fill:var(--text-dim); font-size:13px; }
.m11e .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.m11e .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.m11e .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.m11e .c{ fill:var(--text-faint); font-size:12.5px; }
.m11e .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.m11e .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.m11e .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.m11e .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m11e-a); }
.m11e .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m11e-a); marker-start:url(#m11e-s); }
.m11e .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#m11e-g); }
.m11e .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#m11e-a); }
.m11e .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.m11e .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.m11e .bad{ stroke:var(--text); stroke-width:3; fill:none; }
</style>
<defs><marker id="m11e-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="m11e-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="m11e-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker></defs>
<g class="m11e">
<rect x="20" y="30" width="220" height="80" rx="7" class="pnl"/>
<text x="130.0" y="54.5" text-anchor="middle" class="b">BGP route</text>
<text x="130.0" y="74.5" text-anchor="middle" class="m">192.168.99.0/24</text>
<text x="130.0" y="94.5" text-anchor="middle" class="c">next hop 10.0.0.2, color 10</text>
<rect x="290" y="30" width="160" height="80" rx="7" class="acc"/>
<text x="370.0" y="65.5" text-anchor="middle" class="b">match</text>
<text x="370.0" y="83.5" text-anchor="middle" class="m">(10, 10.0.0.2)</text>
<rect x="500" y="30" width="200" height="80" rx="7" class="dark"/>
<text x="600.0" y="65.5" text-anchor="middle" class="on" style="fill:var(--bg)">push</text>
<text x="600.0" y="83.5" text-anchor="middle" class="on" style="fill:var(--bg)">16013, 16002</text>
<line x1="240" y1="70" x2="288" y2="70" class="ar"/>
<line x1="450" y1="70" x2="498" y2="70" class="ar"/>
<rect x="20" y="150" width="680" height="64" rx="7" class="box"/>
<text x="360.0" y="175.5" text-anchor="middle" class="s">Routes without a color, or with no matching policy, use the normal shortest path.</text>
<text x="360.0" y="197.5" text-anchor="middle" class="c">The color usually arrives as a BGP extended community set by the egress PE or by policy.</text>
</g>
</svg>
<figcaption>Automated steering: the route's color and next hop pick the policy, so no per-route configuration is needed.</figcaption>
</figure>

The usual mechanism is **automated steering**. A BGP route carries a **color** extended community. If the head end has a policy whose color and endpoint match the route's color and BGP next hop, traffic for that route uses the policy's segment list. Routes without a color, or without a matching policy, follow the shortest path. One policy per (color, endpoint) pair therefore serves any number of routes and VPNs, with no per-route configuration.

### Controllers

<figure class="diagram">
<svg viewBox="0 0 720 250" style="width:100%;height:auto;max-width:720px" role="img" aria-label="A path computation element: a controller learns the topology through BGP-LS, computes paths for head ends that ask over PCEP, and can push SR policies to them. The network keeps forwarding if the controller fails." xmlns="http://www.w3.org/2000/svg">
<style>
.m11f{ font-family:var(--sans),sans-serif; }
.m11f .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.m11f .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.m11f .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.m11f .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.m11f .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.m11f .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.m11f .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.m11f .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.m11f .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.m11f .h{ fill:var(--text); font-size:15px; font-weight:700; }
.m11f .b{ fill:var(--text); font-size:13px; font-weight:700; }
.m11f .s{ fill:var(--text-dim); font-size:13px; }
.m11f .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.m11f .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.m11f .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.m11f .c{ fill:var(--text-faint); font-size:12.5px; }
.m11f .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.m11f .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.m11f .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.m11f .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m11f-a); }
.m11f .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m11f-a); marker-start:url(#m11f-s); }
.m11f .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#m11f-g); }
.m11f .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#m11f-a); }
.m11f .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.m11f .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.m11f .bad{ stroke:var(--text); stroke-width:3; fill:none; }
</style>
<defs><marker id="m11f-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="m11f-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="m11f-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker></defs>
<g class="m11f">
<rect x="250" y="20" width="220" height="60" rx="7" class="dark"/>
<text x="360.0" y="45.5" text-anchor="middle" class="on" style="fill:var(--bg)">PCE (controller)</text>
<text x="360.0" y="63.5" text-anchor="middle" class="on" style="fill:var(--bg)">global view, computes paths</text>
<circle cx="90" cy="180" r="24" class="dark"/>
<text x="90" y="184.5" text-anchor="middle" class="on" style="fill:var(--bg)">pe1</text>
<circle cx="270" cy="180" r="24" class="pnl"/>
<text x="270" y="184.5" text-anchor="middle" class="b">p1</text>
<line x1="114" y1="180" x2="246" y2="180" class="lk"/>
<circle cx="450" cy="180" r="24" class="pnl"/>
<text x="450" y="184.5" text-anchor="middle" class="b">p2</text>
<line x1="294" y1="180" x2="426" y2="180" class="lk"/>
<circle cx="630" cy="180" r="24" class="dark"/>
<text x="630" y="184.5" text-anchor="middle" class="on" style="fill:var(--bg)">pe2</text>
<line x1="474" y1="180" x2="606" y2="180" class="lk"/>
<path d="M260 80 L110 154" class="ad"/>
<text x="150" y="110" text-anchor="middle" class="a">PCEP</text>
<path d="M640 156 L460 80" class="ag"/>
<text x="590" y="110" text-anchor="middle" class="c">BGP-LS topology</text>
<text x="360" y="236" text-anchor="middle" class="c">head ends ask for paths (or receive them); the core forwards on SIDs as usual</text>
</g>
</svg>
<figcaption>Centralised path computation for when constraints span many head ends, such as disjoint paths or network-wide optimisation.</figcaption>
</figure>

Some constraints cannot be solved by one head end alone: two paths that must not share any link, or a network-wide bandwidth optimisation. A **path computation element (PCE)** is a controller that learns the full topology (usually through **BGP-LS**), computes paths, and delivers them to head ends with **PCEP**. The head ends still forward on their own; if the controller fails, existing paths keep working. FRR's `pathd` can act as a PCEP client.

## RSVP-TE and SR compared

| | RSVP-TE | SR policy |
|---|---|---|
| Path signalling | PATH and RESV, hop by hop | None: the stack is pushed at the head end |
| State in the core | Per tunnel, refreshed | None beyond node and adjacency SIDs |
| Bandwidth reservation | Built in | Accounted by a controller, if needed |
| Fast reroute | RSVP-TE detours, facility backup | TI-LFA |
| Steering | Static routes, autoroute | Automated steering by BGP color |

## Hands on: an SR policy

The lab `labs/lab6-sr-te` uses the SR core from Chapter 9. pe2 has an extra interface, `svc0`, with the network `192.168.99.0/24`, which it advertises to pe1 over iBGP. Normally pe1 would reach it along the shortest path: pe1, p1, p2, pe2.

<figure class="diagram">
<svg viewBox="0 0 720 280" style="width:100%;height:auto;max-width:720px" role="img" aria-label="Lab 6: without a policy, traffic from pe1 to 192.168.99.0/24 at pe2 follows the shortest path pe1, p1, p2, pe2. With the SR policy of color 10, pe1 pushes 16013 and 16002, and the traffic goes pe1, p3, pe2." xmlns="http://www.w3.org/2000/svg">
<style>
.m11g{ font-family:var(--sans),sans-serif; }
.m11g .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.m11g .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.m11g .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.m11g .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.m11g .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.m11g .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.m11g .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.m11g .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.m11g .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.m11g .h{ fill:var(--text); font-size:15px; font-weight:700; }
.m11g .b{ fill:var(--text); font-size:13px; font-weight:700; }
.m11g .s{ fill:var(--text-dim); font-size:13px; }
.m11g .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.m11g .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.m11g .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.m11g .c{ fill:var(--text-faint); font-size:12.5px; }
.m11g .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.m11g .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.m11g .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.m11g .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m11g-a); }
.m11g .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m11g-a); marker-start:url(#m11g-s); }
.m11g .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#m11g-g); }
.m11g .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#m11g-a); }
.m11g .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.m11g .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.m11g .bad{ stroke:var(--text); stroke-width:3; fill:none; }
</style>
<defs><marker id="m11g-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="m11g-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="m11g-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker></defs>
<g class="m11g">
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
<path d="M114 64 L246 64" class="ag"/>
<path d="M294 64 L426 64" class="ag"/>
<path d="M474 64 L606 64" class="ag"/>
<text x="360" y="40" text-anchor="middle" class="c">default: shortest path (cost 30)</text>
<text x="150" y="190" text-anchor="middle" class="a">policy:</text>
<text x="150" y="208" text-anchor="middle" class="ma">16013, 16002</text>
<text x="560" y="196" text-anchor="middle" class="a">cost 35, but chosen</text>
<text x="360" y="266" text-anchor="middle" class="c">192.168.99.0/24 lives on pe2 (interface svc0)</text>
</g>
</svg>
<figcaption>The lab policy. Only routes carrying color 10 follow it; everything else keeps the shortest path.</figcaption>
</figure>

pe1 runs FRR's `pathd` with a policy that sends traffic through p3, and colours the route from pe2 with color 10:

```text
segment-routing
 traffic-eng
  segment-list VIA-P3
   index 10 mpls label 16013
   index 20 mpls label 16002
  exit
  policy color 10 endpoint 10.0.0.2
   name LOW-LOAD
   binding-sid 15010
   candidate-path preference 100 name VIA-P3 explicit segment-list VIA-P3
  exit
 exit
exit
!
route-map COLOR-10 permit 10
 set sr-te color 10
exit
!
router bgp 65000
 address-family ipv4 unicast
  neighbor 10.0.0.2 route-map COLOR-10 in
```

In a real network pe2 would attach the color as an extended community when advertising the route; here pe1 sets it on receipt, which has the same effect and keeps the lab small.

```bash
cd labs/lab6-sr-te
sudo containerlab deploy -t lab6-sr-te.clab.yml
docker exec -it clab-lab6-sr-te-pe1 vtysh
```

```text
pe1# show sr-te policy detail                 ! policy LOW-LOAD, active candidate VIA-P3
pe1# show bgp ipv4 unicast 192.168.99.0/24     ! the route, with its SR-TE color
pe1# show ip route 192.168.99.0/24             ! resolved through the policy's labels
```

Send traffic from pe1's loopback and capture on p3 to prove the detour:

```bash
sudo ip netns exec clab-lab6-sr-te-p3 tcpdump -nni eth1 mpls &
docker exec clab-lab6-sr-te-pe1 ping -c 3 -I 10.0.0.1 192.168.99.1
```

The packets arrive at p3 carrying label 16002. The first segment, 16013, does not appear on the wire: pe1 resolved it to its neighbour p3, which asked for PHP, so pe1 simply sends the packet to p3 with the rest of the list. Meanwhile a ping from pe1 to pe2's loopback, which has no color, still goes through p1 and p2. As an experiment, remove the `route-map COLOR-10 in` line (then `clear bgp ipv4 unicast 10.0.0.2 soft in`) and watch `192.168.99.0/24` return to the shortest path.

## Summary

- **Traffic engineering** places traffic on chosen paths, which shortest-path routing cannot do.
- **RSVP-TE** computes paths with **CSPF** and signals them with **PATH/RESV**, reserving bandwidth; it keeps **per-tunnel state** in the core.
- An **SR policy** (head end, **color**, endpoint) has prioritised **candidate paths**, explicit or dynamic, and optionally a **binding SID**. The core keeps **no per-path state**.
- **Automated steering** maps BGP routes to policies by **color** and **next hop**.
- A **PCE** computes paths that need a global view and delivers them over **PCEP**. In FRR, **pathd** implements SR policies.
