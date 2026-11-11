# Chapter 6: BGP/MPLS Layer 3 VPNs

The most widely sold MPLS service is the **BGP/MPLS IP VPN** (RFC 4364), usually just called an **L3VPN** or "MPLS VPN". A provider connects the sites of many customers over one shared network, while each customer sees a private IP network of its own, with its own addresses. This chapter builds one from the ground up and runs it in the lab.

## The problem

Customers number their sites with private address space, and they all pick the same popular ranges.

<figure class="diagram">
<svg viewBox="0 0 720 250" style="width:100%;height:auto;max-width:720px" role="img" aria-label="Two customers, RED and BLUE, both use 192.168.1.0/24 at site A and 192.168.2.0/24 at site B. A single routing table on the PE cannot hold both: the routes collide." xmlns="http://www.w3.org/2000/svg">
<style>
.m6a{ font-family:var(--sans),sans-serif; }
.m6a .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.m6a .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.m6a .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.m6a .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.m6a .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.m6a .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.m6a .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.m6a .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.m6a .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.m6a .h{ fill:var(--text); font-size:15px; font-weight:700; }
.m6a .b{ fill:var(--text); font-size:13px; font-weight:700; }
.m6a .s{ fill:var(--text-dim); font-size:13px; }
.m6a .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.m6a .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.m6a .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.m6a .c{ fill:var(--text-faint); font-size:12.5px; }
.m6a .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.m6a .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.m6a .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.m6a .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m6a-a); }
.m6a .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m6a-a); marker-start:url(#m6a-s); }
.m6a .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#m6a-g); }
.m6a .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#m6a-a); }
.m6a .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.m6a .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.m6a .bad{ stroke:var(--text); stroke-width:3; fill:none; }
</style>
<defs><marker id="m6a-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="m6a-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="m6a-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker></defs>
<g class="m6a">
<rect x="10" y="36" width="140" height="48" rx="7" class="acc"/>
<text x="80.0" y="55.5" text-anchor="middle" class="b">RED A</text>
<text x="80.0" y="73.5" text-anchor="middle" class="m">192.168.1.0/24</text>
<rect x="10" y="156" width="140" height="48" rx="7" class="pnl"/>
<text x="80.0" y="175.5" text-anchor="middle" class="b">BLUE A</text>
<text x="80.0" y="193.5" text-anchor="middle" class="m">192.168.1.0/24</text>
<rect x="570" y="36" width="140" height="48" rx="7" class="acc"/>
<text x="640.0" y="55.5" text-anchor="middle" class="b">RED B</text>
<text x="640.0" y="73.5" text-anchor="middle" class="m">192.168.2.0/24</text>
<rect x="570" y="156" width="140" height="48" rx="7" class="pnl"/>
<text x="640.0" y="175.5" text-anchor="middle" class="b">BLUE B</text>
<text x="640.0" y="193.5" text-anchor="middle" class="m">192.168.2.0/24</text>
<rect x="250" y="70" width="220" height="100" rx="7" class="box"/>
<text x="360.0" y="106.5" text-anchor="middle" class="b">one routing table</text>
<text x="360.0" y="124.5" text-anchor="middle" class="m">192.168.1.0/24 via RED ?</text>
<text x="360.0" y="142.5" text-anchor="middle" class="m">192.168.1.0/24 via BLUE ?</text>
<line x1="150" y1="60" x2="248" y2="100" class="lk"/>
<line x1="150" y1="180" x2="248" y2="140" class="lk"/>
<line x1="472" y1="100" x2="568" y2="60" class="lk"/>
<line x1="472" y1="140" x2="568" y2="180" class="lk"/>
<text x="360" y="220" text-anchor="middle" class="c">private addresses overlap between customers, so one table cannot serve them all</text>
</g>
</svg>
<figcaption>The VPN problem. Customers pick their own private addresses, and they collide.</figcaption>
</figure>

A single routing table cannot say "192.168.1.0/24 is at RED's site A" and "192.168.1.0/24 is at BLUE's site A" at the same time. Even if addresses did not overlap, customers must not be able to reach each other. The solution needs three parts: separate tables on the PEs, a way to carry overlapping routes between PEs, and a way to keep the traffic apart in the core. MPLS VPNs use a VRF, MP-BGP, and a label stack for the three, respectively.

## VRFs: separate tables on the PE

A **VRF** (virtual routing and forwarding instance) is a separate routing table, together with the interfaces bound to it.

<figure class="diagram">
<svg viewBox="0 0 720 270" style="width:100%;height:auto;max-width:720px" role="img" aria-label="A PE with two VRFs: VRF RED holds RED&#x27;s routes and the interface to CE1, VRF BLUE holds BLUE&#x27;s routes and the interface to CE3, and the global table holds the core routes and loopbacks. Each VRF is a separate routing table." xmlns="http://www.w3.org/2000/svg">
<style>
.m6b{ font-family:var(--sans),sans-serif; }
.m6b .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.m6b .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.m6b .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.m6b .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.m6b .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.m6b .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.m6b .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.m6b .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.m6b .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.m6b .h{ fill:var(--text); font-size:15px; font-weight:700; }
.m6b .b{ fill:var(--text); font-size:13px; font-weight:700; }
.m6b .s{ fill:var(--text-dim); font-size:13px; }
.m6b .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.m6b .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.m6b .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.m6b .c{ fill:var(--text-faint); font-size:12.5px; }
.m6b .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.m6b .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.m6b .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.m6b .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m6b-a); }
.m6b .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m6b-a); marker-start:url(#m6b-s); }
.m6b .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#m6b-g); }
.m6b .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#m6b-a); }
.m6b .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.m6b .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.m6b .bad{ stroke:var(--text); stroke-width:3; fill:none; }
</style>
<defs><marker id="m6b-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="m6b-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="m6b-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker></defs>
<g class="m6b">
<rect x="170" y="20" width="380" height="236" rx="7" class="out"/>
<text x="360" y="44" text-anchor="middle" class="h">pe1</text>
<rect x="190" y="60" width="340" height="52" rx="7" class="acc"/>
<text x="360.0" y="81.5" text-anchor="middle" class="b">VRF RED (table 10)</text>
<text x="360.0" y="99.5" text-anchor="middle" class="m">192.168.1.0/24 via ce1 · eth3</text>
<rect x="190" y="124" width="340" height="52" rx="7" class="pnl"/>
<text x="360.0" y="145.5" text-anchor="middle" class="b">VRF BLUE (table 20)</text>
<text x="360.0" y="163.5" text-anchor="middle" class="m">192.168.1.0/24 via ce3 · eth4</text>
<rect x="190" y="188" width="340" height="52" rx="7" class="box"/>
<text x="360.0" y="209.5" text-anchor="middle" class="b">global table</text>
<text x="360.0" y="227.5" text-anchor="middle" class="m">10.0.0.0/24 loopbacks, core links</text>
<circle cx="60" cy="86" r="24" class="acc"/>
<text x="60" y="90.5" text-anchor="middle" class="b">ce1</text>
<circle cx="60" cy="150" r="24" class="pnl"/>
<text x="60" y="154.5" text-anchor="middle" class="b">ce3</text>
<line x1="84" y1="86" x2="188" y2="86" class="lk"/>
<line x1="84" y1="150" x2="188" y2="150" class="lk"/>
<line x1="532" y1="214" x2="640" y2="214" class="lk"/>
<text x="660" y="218" text-anchor="middle" class="s">core</text>
<text x="110" y="76" text-anchor="middle" class="c">eth3</text>
<text x="110" y="140" text-anchor="middle" class="c">eth4</text>
<text x="590" y="204" text-anchor="middle" class="c">eth1, eth2</text>
</g>
</svg>
<figcaption>A VRF is a separate routing table plus the interfaces bound to it. The same prefix can live in many VRFs at once.</figcaption>
</figure>

A PE has one VRF per customer it serves at that location, plus the global table for the core. A packet arriving on `eth3` is looked up only in VRF RED; a packet on `eth4` only in VRF BLUE. CE routers are unaware of any of this. They simply route to their PE.

On Linux, a VRF is a network device of type `vrf` tied to a routing table number. Interfaces are enslaved to it:

```bash
ip link add RED type vrf table 10
ip link set RED up
ip link set eth3 master RED
ip route show vrf RED          # the VRF's table
```

FRR picks VRFs up from the kernel and lets each protocol run per VRF.

VRFs alone give you separation on one router, which is sometimes called **VRF-lite**. To join sites across the provider network, routes and traffic must cross the core while staying separate.

## Route distinguishers and route targets

PEs exchange customer routes over iBGP, using a special address family, **VPNv4** (AFI 1, SAFI 128; VPNv6 for IPv6). Two extra values attach to every VPN route, and they do different jobs.

<figure class="diagram">
<svg viewBox="0 0 720 290" style="width:100%;height:auto;max-width:720px" role="img" aria-label="Route distinguisher and route target. The RD is prepended to a customer prefix to make a unique VPNv4 route: 10.0.0.1:10 plus 192.168.1.0/24 differs from 10.0.0.1:20 plus 192.168.1.0/24. The RT is an extended community attached to the route; a PE imports the route into every VRF whose import list contains that RT." xmlns="http://www.w3.org/2000/svg">
<style>
.m6c{ font-family:var(--sans),sans-serif; }
.m6c .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.m6c .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.m6c .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.m6c .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.m6c .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.m6c .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.m6c .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.m6c .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.m6c .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.m6c .h{ fill:var(--text); font-size:15px; font-weight:700; }
.m6c .b{ fill:var(--text); font-size:13px; font-weight:700; }
.m6c .s{ fill:var(--text-dim); font-size:13px; }
.m6c .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.m6c .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.m6c .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.m6c .c{ fill:var(--text-faint); font-size:12.5px; }
.m6c .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.m6c .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.m6c .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.m6c .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m6c-a); }
.m6c .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m6c-a); marker-start:url(#m6c-s); }
.m6c .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#m6c-g); }
.m6c .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#m6c-a); }
.m6c .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.m6c .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.m6c .bad{ stroke:var(--text); stroke-width:3; fill:none; }
</style>
<defs><marker id="m6c-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="m6c-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="m6c-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker></defs>
<g class="m6c">
<text x="20" y="28" text-anchor="start" class="h">RD: make the route unique</text>
<rect x="20" y="44" width="150" height="32" rx="3" class="acc"/>
<text x="95.0" y="64.5" text-anchor="middle" class="m">RD 10.0.0.1:10</text>
<rect x="170" y="44" width="150" height="32" rx="3" class="box"/>
<text x="245.0" y="64.5" text-anchor="middle" class="m">192.168.1.0/24</text>
<rect x="20" y="84" width="150" height="32" rx="3" class="pnl"/>
<text x="95.0" y="104.5" text-anchor="middle" class="m">RD 10.0.0.1:20</text>
<rect x="170" y="84" width="150" height="32" rx="3" class="box"/>
<text x="245.0" y="104.5" text-anchor="middle" class="m">192.168.1.0/24</text>
<text x="170" y="140" text-anchor="middle" class="c">two different 96-bit VPNv4 prefixes</text>
<text x="380" y="28" text-anchor="start" class="h">RT: decide who imports it</text>
<rect x="380" y="44" width="140" height="72" rx="7" class="acc"/>
<text x="450.0" y="75.5" text-anchor="middle" class="b">VPNv4 route</text>
<text x="450.0" y="93.5" text-anchor="middle" class="m">RT 65000:10</text>
<rect x="580" y="30" width="130" height="44" rx="7" class="acc"/>
<text x="645.0" y="47.5" text-anchor="middle" class="b">VRF RED</text>
<text x="645.0" y="65.5" text-anchor="middle" class="m">import 65000:10</text>
<rect x="580" y="88" width="130" height="44" rx="7" class="pnl"/>
<text x="645.0" y="105.5" text-anchor="middle" class="b">VRF BLUE</text>
<text x="645.0" y="123.5" text-anchor="middle" class="m">import 65000:20</text>
<line x1="520" y1="70" x2="578" y2="54" class="ar"/>
<line x1="520" y1="90" x2="578" y2="108" class="ag"/>
<text x="560" y="152" text-anchor="middle" class="c">no match: ignored</text>
<rect x="20" y="196" width="690" height="70" rx="7" class="pnl"/>
<text x="365.0" y="226.5" text-anchor="middle" class="s">The RD only makes routes unique; it carries no policy.</text>
<text x="365.0" y="244.5" text-anchor="middle" class="s">The RT is the policy: export RTs are attached when a route leaves a VRF, import RTs select routes entering one.</text>
</g>
</svg>
<figcaption>Two separate jobs. RDs keep overlapping prefixes apart in BGP; RTs decide which VRFs a route lands in.</figcaption>
</figure>

- The **route distinguisher (RD)** is a 64-bit value prepended to the IPv4 prefix to make a 96-bit VPNv4 prefix. RED's `192.168.1.0/24` and BLUE's `192.168.1.0/24` get different RDs, so BGP sees two different prefixes and keeps both. RDs are written `ASN:number` or `IP:number`; using the PE's router ID and a VRF number, as the labs do (`10.0.0.1:10`), makes each RD unique per PE and VRF.
- The **route target (RT)** is a BGP extended community. When a route leaves a VRF, the VRF's **export** RTs are attached. A PE receiving the route places it into every local VRF whose **import** RTs include one of them. RTs are the VPN membership policy.

Keeping the two concepts apart is the key to understanding L3VPN design. The RD carries **no policy** at all: two VRFs with different RDs can still import each other's routes if their RTs say so. Chapter 7 uses RTs to build topologies far more interesting than "each customer talks to itself".

## The control plane

<figure class="diagram">
<svg viewBox="0 0 720 300" style="width:100%;height:auto;max-width:720px" role="img" aria-label="L3VPN control plane for RED: ce1&#x27;s subnet arrives in pe1&#x27;s VRF RED; pe1 exports it as VPNv4 route 10.0.0.1:10:192.168.1.0/24 with RT 65000:10, next hop 10.0.0.1 and VPN label 80; it sends this over iBGP to pe2; pe2 imports it into its VRF RED because the RT matches, and ce2 can now reach it." xmlns="http://www.w3.org/2000/svg">
<style>
.m6d{ font-family:var(--sans),sans-serif; }
.m6d .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.m6d .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.m6d .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.m6d .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.m6d .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.m6d .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.m6d .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.m6d .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.m6d .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.m6d .h{ fill:var(--text); font-size:15px; font-weight:700; }
.m6d .b{ fill:var(--text); font-size:13px; font-weight:700; }
.m6d .s{ fill:var(--text-dim); font-size:13px; }
.m6d .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.m6d .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.m6d .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.m6d .c{ fill:var(--text-faint); font-size:12.5px; }
.m6d .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.m6d .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.m6d .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.m6d .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m6d-a); }
.m6d .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m6d-a); marker-start:url(#m6d-s); }
.m6d .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#m6d-g); }
.m6d .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#m6d-a); }
.m6d .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.m6d .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.m6d .bad{ stroke:var(--text); stroke-width:3; fill:none; }
</style>
<defs><marker id="m6d-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="m6d-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="m6d-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker></defs>
<g class="m6d">
<circle cx="50" cy="90" r="24" class="acc"/>
<text x="50" y="94.5" text-anchor="middle" class="b">ce1</text>
<circle cx="670" cy="90" r="24" class="acc"/>
<text x="670" y="94.5" text-anchor="middle" class="b">ce2</text>
<rect x="100" y="60" width="150" height="60" rx="7" class="acc"/>
<text x="175.0" y="85.5" text-anchor="middle" class="b">pe1 VRF RED</text>
<text x="175.0" y="103.5" text-anchor="middle" class="m">192.168.1.0/24</text>
<rect x="470" y="60" width="150" height="60" rx="7" class="acc"/>
<text x="545.0" y="85.5" text-anchor="middle" class="b">pe2 VRF RED</text>
<text x="545.0" y="103.5" text-anchor="middle" class="m">192.168.1.0/24</text>
<line x1="74" y1="90" x2="98" y2="90" class="ar"/>
<line x1="622" y1="90" x2="646" y2="90" class="ag"/>
<path d="M250 80 C 310 30, 410 30, 468 80" class="ad"/>
<text x="360" y="34" text-anchor="middle" class="a">MP-BGP, address family VPNv4</text>
<rect x="170" y="150" width="380" height="120" rx="7" class="pnl"/>
<text x="360.0" y="176.5" text-anchor="middle" class="b">VPNv4 update</text>
<text x="360.0" y="195.5" text-anchor="middle" class="m">prefix  10.0.0.1:10:192.168.1.0/24</text>
<text x="360.0" y="214.5" text-anchor="middle" class="m">next hop  10.0.0.1</text>
<text x="360.0" y="233.5" text-anchor="middle" class="m">label  80   (pe1's VPN label)</text>
<text x="360.0" y="252.5" text-anchor="middle" class="ma">RT  65000:10</text>
<text x="85" y="160" text-anchor="middle" class="s">1. export</text>
<text x="85" y="176" text-anchor="middle" class="c">add RD, RT,</text>
<text x="85" y="192" text-anchor="middle" class="c">label</text>
<text x="635" y="160" text-anchor="middle" class="s">2. import</text>
<text x="635" y="176" text-anchor="middle" class="c">RT matches</text>
<text x="635" y="192" text-anchor="middle" class="c">VRF RED</text>
</g>
</svg>
<figcaption>Exporting and importing. The VPN label travels inside the BGP update; the core never sees any of this.</figcaption>
</figure>

Follow RED's site A prefix from ce1 to ce2:

1. pe1 learns `192.168.1.0/24` in VRF RED, from a static route, a connected interface, or a routing protocol with ce1.
2. pe1 **exports** it: it adds the RD `10.0.0.1:10`, the export RT `65000:10`, and a **VPN label** that it allocates locally (say 80), and sets the next hop to its own loopback, `10.0.0.1`.
3. The VPNv4 route crosses the provider network by iBGP to pe2 (in a real network, through route reflectors).
4. pe2 sees RT `65000:10`, which VRF RED imports, so it **imports** the route into VRF RED: `192.168.1.0/24` via `10.0.0.1`, with label 80.
5. pe2 advertises the route on to ce2.

None of this involves the P routers. They have no VRFs, no VPN routes, and no BGP.

## The data plane

<figure class="diagram">
<svg viewBox="0 0 720 280" style="width:100%;height:auto;max-width:720px" role="img" aria-label="L3VPN data plane: ce1 sends an IP packet to 192.168.2.1; pe1 looks it up in VRF RED and pushes two labels, the VPN label 81 from pe2 at the bottom and the LDP label 22 toward pe2&#x27;s loopback on top; p1 swaps 22 to 18; p2 pops the top label (PHP); pe2 receives the packet with only the VPN label, which maps to VRF RED, and forwards the IP packet to ce2." xmlns="http://www.w3.org/2000/svg">
<style>
.m6e{ font-family:var(--sans),sans-serif; }
.m6e .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.m6e .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.m6e .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.m6e .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.m6e .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.m6e .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.m6e .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.m6e .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.m6e .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.m6e .h{ fill:var(--text); font-size:15px; font-weight:700; }
.m6e .b{ fill:var(--text); font-size:13px; font-weight:700; }
.m6e .s{ fill:var(--text-dim); font-size:13px; }
.m6e .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.m6e .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.m6e .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.m6e .c{ fill:var(--text-faint); font-size:12.5px; }
.m6e .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.m6e .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.m6e .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.m6e .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m6e-a); }
.m6e .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m6e-a); marker-start:url(#m6e-s); }
.m6e .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#m6e-g); }
.m6e .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#m6e-a); }
.m6e .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.m6e .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.m6e .bad{ stroke:var(--text); stroke-width:3; fill:none; }
</style>
<defs><marker id="m6e-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="m6e-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="m6e-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker></defs>
<g class="m6e">
<circle cx="50" cy="50" r="24" class="acc"/>
<text x="50" y="54.5" text-anchor="middle" class="b">ce1</text>
<circle cx="170" cy="50" r="24" class="dark"/>
<text x="170" y="54.5" text-anchor="middle" class="on" style="fill:var(--bg)">pe1</text>
<circle cx="300" cy="50" r="24" class="pnl"/>
<text x="300" y="54.5" text-anchor="middle" class="b">p1</text>
<circle cx="420" cy="50" r="24" class="pnl"/>
<text x="420" y="54.5" text-anchor="middle" class="b">p2</text>
<circle cx="550" cy="50" r="24" class="dark"/>
<text x="550" y="54.5" text-anchor="middle" class="on" style="fill:var(--bg)">pe2</text>
<circle cx="670" cy="50" r="24" class="acc"/>
<text x="670" y="54.5" text-anchor="middle" class="b">ce2</text>
<line x1="74" y1="50" x2="146" y2="50" class="lk"/>
<line x1="194" y1="50" x2="276" y2="50" class="lk"/>
<line x1="324" y1="50" x2="396" y2="50" class="lk"/>
<line x1="444" y1="50" x2="526" y2="50" class="lk"/>
<line x1="574" y1="50" x2="646" y2="50" class="lk"/>
<rect x="70" y="100" width="40" height="28" rx="3" class="box"/>
<text x="90.0" y="118.5" text-anchor="middle" class="m">IP</text>
<rect x="195" y="100" width="32" height="28" rx="3" class="acc"/>
<text x="211.0" y="118.5" text-anchor="middle" class="m">22</text>
<rect x="227" y="100" width="32" height="28" rx="3" class="dark"/>
<text x="243.0" y="118.5" text-anchor="middle" class="on" style="fill:var(--bg)">81</text>
<rect x="259" y="100" width="40" height="28" rx="3" class="box"/>
<text x="279.0" y="118.5" text-anchor="middle" class="m">IP</text>
<rect x="320" y="100" width="32" height="28" rx="3" class="acc"/>
<text x="336.0" y="118.5" text-anchor="middle" class="m">18</text>
<rect x="352" y="100" width="32" height="28" rx="3" class="dark"/>
<text x="368.0" y="118.5" text-anchor="middle" class="on" style="fill:var(--bg)">81</text>
<rect x="384" y="100" width="40" height="28" rx="3" class="box"/>
<text x="404.0" y="118.5" text-anchor="middle" class="m">IP</text>
<rect x="452" y="100" width="32" height="28" rx="3" class="dark"/>
<text x="468.0" y="118.5" text-anchor="middle" class="on" style="fill:var(--bg)">81</text>
<rect x="484" y="100" width="40" height="28" rx="3" class="box"/>
<text x="504.0" y="118.5" text-anchor="middle" class="m">IP</text>
<rect x="590" y="100" width="40" height="28" rx="3" class="box"/>
<text x="610.0" y="118.5" text-anchor="middle" class="m">IP</text>
<text x="170" y="162" text-anchor="middle" class="c">VRF lookup,</text>
<text x="170" y="178" text-anchor="middle" class="c">push 81, push 22</text>
<text x="300" y="162" text-anchor="middle" class="c">swap 22 to 18</text>
<text x="420" y="162" text-anchor="middle" class="c">PHP: pop 18</text>
<text x="550" y="162" text-anchor="middle" class="c">81 means VRF RED:</text>
<text x="550" y="178" text-anchor="middle" class="c">pop, IP lookup</text>
<rect x="170" y="212" width="180" height="40" rx="7" class="acc"/>
<text x="260.0" y="236.5" text-anchor="middle" class="s">transport label (LDP)</text>
<rect x="380" y="212" width="180" height="40" rx="7" class="dark"/>
<text x="470.0" y="236.5" text-anchor="middle" class="on" style="fill:var(--bg)">VPN label (BGP)</text>
</g>
</svg>
<figcaption>Two labels, two jobs. The top label gets the packet to pe2; the bottom label tells pe2 which VRF it belongs to.</figcaption>
</figure>

When ce1 sends a packet to `192.168.2.1`:

1. pe1 receives it on `eth3`, so it looks it up in **VRF RED** and finds the route from pe2: next hop `10.0.0.2`, VPN label 81.
2. pe1 pushes **two labels**: the VPN label 81 at the bottom, and on top the **transport label** that LDP (or Segment Routing) provides for `10.0.0.2`.
3. The P routers switch on the top label only. With PHP, the router before pe2 pops it.
4. pe2 receives the packet with the VPN label on top. Label 81 is its own label, and it means "this packet belongs to VRF RED". It pops the label and routes the packet in VRF RED, toward ce2.

This is the BGP-free core of Chapter 5 with one more label. The transport label gets the packet to the right PE; the VPN label tells that PE which customer the packet belongs to. The same packet from ce3 to `192.168.2.1` takes the same transport label but a different VPN label, and arrives at ce4.

### How many VPN labels?

<figure class="diagram">
<svg viewBox="0 0 720 230" style="width:100%;height:auto;max-width:720px" role="img" aria-label="VPN label allocation modes: per prefix uses one label for each VPN route, most labels; per VRF uses one label per VRF and needs an IP lookup at the egress; per CE or next hop uses one label per attached neighbour and forwards without an IP lookup." xmlns="http://www.w3.org/2000/svg">
<style>
.m6f{ font-family:var(--sans),sans-serif; }
.m6f .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.m6f .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.m6f .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.m6f .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.m6f .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.m6f .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.m6f .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.m6f .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.m6f .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.m6f .h{ fill:var(--text); font-size:15px; font-weight:700; }
.m6f .b{ fill:var(--text); font-size:13px; font-weight:700; }
.m6f .s{ fill:var(--text-dim); font-size:13px; }
.m6f .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.m6f .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.m6f .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.m6f .c{ fill:var(--text-faint); font-size:12.5px; }
.m6f .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.m6f .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.m6f .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.m6f .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m6f-a); }
.m6f .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m6f-a); marker-start:url(#m6f-s); }
.m6f .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#m6f-g); }
.m6f .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#m6f-a); }
.m6f .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.m6f .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.m6f .bad{ stroke:var(--text); stroke-width:3; fill:none; }
</style>
<defs><marker id="m6f-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="m6f-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="m6f-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker></defs>
<g class="m6f">
<rect x="20" y="30" width="216" height="100" rx="7" class="box"/>
<text x="128.0" y="62.5" text-anchor="middle" class="b">per prefix</text>
<text x="128.0" y="84.5" text-anchor="middle" class="s">one label per route</text>
<text x="128.0" y="106.5" text-anchor="middle" class="c">most labels, no egress IP lookup</text>
<rect x="252" y="30" width="216" height="100" rx="7" class="pnl"/>
<text x="360.0" y="62.5" text-anchor="middle" class="b">per CE / next hop</text>
<text x="360.0" y="84.5" text-anchor="middle" class="s">one label per neighbour</text>
<text x="360.0" y="106.5" text-anchor="middle" class="c">few labels, no egress IP lookup</text>
<rect x="484" y="30" width="216" height="100" rx="7" class="acc"/>
<text x="592.0" y="62.5" text-anchor="middle" class="b">per VRF</text>
<text x="592.0" y="84.5" text-anchor="middle" class="s">one label per VRF</text>
<text x="592.0" y="106.5" text-anchor="middle" class="s">fewest labels, egress IP lookup</text>
<text x="360" y="170" text-anchor="middle" class="s">FRR's  label vpn export auto  allocates one label per VRF</text>
<text x="360" y="196" text-anchor="middle" class="c">and the egress PE does an IP lookup in that VRF</text>
</g>
</svg>
<figcaption>How many VPN labels to allocate is a trade-off between label space and work at the egress PE.</figcaption>
</figure>

A PE can allocate one VPN label per prefix, per CE (next hop), or per VRF. Per-prefix labels let the egress forward without an IP lookup, at the cost of many labels; per-VRF labels need an IP lookup in the VRF at the egress, but use one label per customer per PE. FRR's `label vpn export auto` allocates one label per VRF, and the Linux kernel implements it as "pop the label, then look up in the VRF's table".

## Configuring L3VPN in FRR

The VPN configuration on pe1 has two parts. The default BGP instance carries VPNv4 routes to the other PE:

```text
router bgp 65000
 bgp router-id 10.0.0.1
 no bgp default ipv4-unicast
 neighbor 10.0.0.2 remote-as 65000
 neighbor 10.0.0.2 update-source lo
 !
 address-family ipv4 vpn
  neighbor 10.0.0.2 activate
 exit-address-family
exit
```

A BGP instance per VRF holds that customer's routes and controls how they are exported and imported:

```text
router bgp 65000 vrf RED
 bgp router-id 10.0.0.1
 !
 address-family ipv4 unicast
  redistribute static              ! routes toward the customer site
  redistribute connected
  label vpn export auto            ! allocate a VPN label for this VRF
  rd vpn export 10.0.0.1:10        ! route distinguisher
  rt vpn both 65000:10             ! export and import route target
  export vpn                       ! copy VRF routes into VPNv4
  import vpn                       ! copy matching VPNv4 routes into the VRF
 exit-address-family
exit
```

`rt vpn both` is shorthand for the same RT in both directions; `rt vpn import` and `rt vpn export` set them separately. The static route to the customer lives in the VRF as well:

```text
vrf RED
 ip route 192.168.1.0/24 172.16.1.2
exit-vrf
```

## Hands on: two customers, one network

The lab `labs/lab4-l3vpn` is the LDP core with VRFs RED and BLUE on both PEs and the four CE sites from Chapter 3. RED and BLUE use identical addresses. The topology file creates the VRF devices with `ip link` commands, and the PE configurations contain everything shown above.

```bash
cd labs/lab4-l3vpn
sudo containerlab deploy -t lab4-l3vpn.clab.yml
```

Give BGP a few seconds, then test both customers with the very same addresses:

```bash
docker exec clab-lab4-l3vpn-ce1 ping -c 3 -I 192.168.1.1 192.168.2.1    # RED: reaches ce2
docker exec clab-lab4-l3vpn-ce3 ping -c 3 -I 192.168.1.1 192.168.2.1    # BLUE: reaches ce4
```

Both work, and each reaches its own site. Now look at the control plane on pe1:

```text
pe1# show bgp ipv4 vpn                  ! all VPNv4 routes, grouped by RD
pe1# show bgp vrf RED ipv4 unicast       ! RED's view after import
pe1# show ip route vrf RED               ! RED's routing table
pe1# show ip route vrf BLUE              ! same prefixes, different table
```

In `show bgp ipv4 vpn`, you will see `192.168.2.0/24` twice: once under RD `10.0.0.2:10` and once under `10.0.0.2:20`. In the VRF routing tables, the remote routes point at `10.0.0.2` with **two labels**: the transport label first, then the VPN label. Compare the VPN labels for RED and BLUE: they differ.

At pe2, each VPN label maps to a VRF:

```text
pe2# show mpls table
```

Look for two entries whose action is to pop and look up in a VRF (RED and BLUE). Watch the label stack on the wire on the core link into pe2:

```bash
sudo ip netns exec clab-lab4-l3vpn-p2 tcpdump -nni eth2 mpls
```

Because of PHP, packets arriving at pe2 from p2 carry only the VPN label. Capture on p1's interfaces instead to see both labels.

### Experiment: what does the RD do?

Change BLUE's RD on pe1 to be the same as RED's (`rd vpn export 10.0.0.1:10` under `router bgp 65000 vrf BLUE`). Now RED's and BLUE's `192.168.1.0/24` become the same VPNv4 prefix, and BGP keeps only one of them as best. That breaks one customer. Unique RDs per VRF, per PE, avoid this. Restore the original value afterward.

## Summary

- An **L3VPN** gives each customer a private routed network over a shared MPLS core.
- **VRFs** keep each customer's routes in a separate table on the PE.
- PEs exchange **VPNv4** routes over MP-BGP. The **RD** makes overlapping prefixes unique; the **RT** decides which VRFs import a route.
- Traffic carries **two labels**: a **transport** label to the egress PE, and a **VPN** label that selects the VRF there.
- P routers know nothing about VPNs. In FRR, `label vpn export auto`, `rd vpn export`, `rt vpn`, `export vpn` and `import vpn` configure a VRF for L3VPN.
