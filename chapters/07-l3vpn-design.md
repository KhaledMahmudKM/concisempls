# Chapter 7: L3VPN Design

Chapter 6 built the simplest VPN: every site of a customer talks to every other site of the same customer. Real VPN services are more varied. Some customers want all traffic to pass through a central firewall; some need access to a provider's shared services; most need Internet access; and a few span more than one provider. This chapter covers the design patterns that make these possible, and the scaling techniques that keep a network with thousands of VPNs manageable.

## Scaling BGP: route reflectors

Every PE must exchange VPN routes with every other PE that shares a customer. A full iBGP mesh grows with the square of the number of PEs.

<figure class="diagram">
<svg viewBox="0 0 720 260" style="width:100%;height:auto;max-width:720px" role="img" aria-label="Full mesh versus route reflector: with six PEs a full iBGP mesh needs fifteen sessions; with a route reflector each PE has one session to the reflector, which can filter VPN routes by route target." xmlns="http://www.w3.org/2000/svg">
<style>
.m7a{ font-family:var(--sans),sans-serif; }
.m7a .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.m7a .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.m7a .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.m7a .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.m7a .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.m7a .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.m7a .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.m7a .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.m7a .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.m7a .h{ fill:var(--text); font-size:15px; font-weight:700; }
.m7a .b{ fill:var(--text); font-size:13px; font-weight:700; }
.m7a .s{ fill:var(--text-dim); font-size:13px; }
.m7a .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.m7a .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.m7a .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.m7a .c{ fill:var(--text-faint); font-size:12.5px; }
.m7a .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.m7a .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.m7a .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.m7a .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m7a-a); }
.m7a .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m7a-a); marker-start:url(#m7a-s); }
.m7a .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#m7a-g); }
.m7a .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#m7a-a); }
.m7a .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.m7a .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.m7a .bad{ stroke:var(--text); stroke-width:3; fill:none; }
</style>
<defs><marker id="m7a-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="m7a-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="m7a-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker></defs>
<g class="m7a">
<line x1="180.0" y1="40.0" x2="257.9" y2="85.0" class="ln"/>
<line x1="180.0" y1="40.0" x2="257.9" y2="175.0" class="ln"/>
<line x1="180.0" y1="40.0" x2="180.0" y2="220.0" class="ln"/>
<line x1="180.0" y1="40.0" x2="102.1" y2="175.0" class="ln"/>
<line x1="180.0" y1="40.0" x2="102.1" y2="85.0" class="ln"/>
<line x1="257.9" y1="85.0" x2="257.9" y2="175.0" class="ln"/>
<line x1="257.9" y1="85.0" x2="180.0" y2="220.0" class="ln"/>
<line x1="257.9" y1="85.0" x2="102.1" y2="175.0" class="ln"/>
<line x1="257.9" y1="85.0" x2="102.1" y2="85.0" class="ln"/>
<line x1="257.9" y1="175.0" x2="180.0" y2="220.0" class="ln"/>
<line x1="257.9" y1="175.0" x2="102.1" y2="175.0" class="ln"/>
<line x1="257.9" y1="175.0" x2="102.1" y2="85.0" class="ln"/>
<line x1="180.0" y1="220.0" x2="102.1" y2="175.0" class="ln"/>
<line x1="180.0" y1="220.0" x2="102.1" y2="85.0" class="ln"/>
<line x1="102.1" y1="175.0" x2="102.1" y2="85.0" class="ln"/>
<circle cx="180.0" cy="40.0" r="16" class="dark"/>
<circle cx="257.9" cy="85.0" r="16" class="dark"/>
<circle cx="257.9" cy="175.0" r="16" class="dark"/>
<circle cx="180.0" cy="220.0" r="16" class="dark"/>
<circle cx="102.1" cy="175.0" r="16" class="dark"/>
<circle cx="102.1" cy="85.0" r="16" class="dark"/>
<text x="180" y="250" text-anchor="middle" class="s">full mesh: n(n-1)/2 = 15 sessions</text>
<line x1="540.0" y1="103.0" x2="540.0" y2="57.0" class="lk"/>
<circle cx="540.0" cy="40.0" r="16" class="dark"/>
<line x1="563.4" y1="116.5" x2="603.2" y2="93.5" class="lk"/>
<circle cx="617.9" cy="85.0" r="16" class="dark"/>
<line x1="563.4" y1="143.5" x2="603.2" y2="166.5" class="lk"/>
<circle cx="617.9" cy="175.0" r="16" class="dark"/>
<line x1="540.0" y1="157.0" x2="540.0" y2="203.0" class="lk"/>
<circle cx="540.0" cy="220.0" r="16" class="dark"/>
<line x1="516.6" y1="143.5" x2="476.8" y2="166.5" class="lk"/>
<circle cx="462.1" cy="175.0" r="16" class="dark"/>
<line x1="516.6" y1="116.5" x2="476.8" y2="93.5" class="lk"/>
<circle cx="462.1" cy="85.0" r="16" class="dark"/>
<circle cx="540" cy="130" r="26" class="acc"/>
<text x="540" y="135" text-anchor="middle" class="b">RR</text>
<text x="540" y="250" text-anchor="middle" class="s">route reflector: 6 sessions</text>
</g>
</svg>
<figcaption>VPN networks almost always use route reflectors. The reflectors carry VPNv4 routes but need not forward any traffic.</figcaption>
</figure>

VPN networks therefore use **route reflectors (RRs)**, usually two or more for redundancy. Each PE has a session to each RR. The RRs carry VPNv4 routes but never forward VPN traffic: the BGP next hop stays the originating PE's loopback, so traffic still flows directly between PEs. Reflectors can therefore be virtual machines or small routers placed anywhere.

Two further techniques keep the amount of routing information down:

- **Route target constraint** (RFC 4684). Each PE tells the reflector which RTs it imports, and the reflector sends it only routes carrying those RTs. Without it, every PE receives every VPN route and throws most of them away.
- **Per-VRF route limits.** A maximum number of routes per VRF protects the PE from a customer that accidentally redistributes a full table.

## Topologies with route targets

Because import and export RTs are independent, a provider can build almost any topology for a customer.

### Any-to-any

Every VRF of a customer imports and exports the same RT. This is the default from Chapter 6.

### Hub and spoke

<figure class="diagram">
<svg viewBox="0 0 720 290" style="width:100%;height:auto;max-width:720px" role="img" aria-label="Hub and spoke with route targets: each spoke VRF exports RT SPOKE and imports only RT HUB; the hub VRF imports RT SPOKE and exports RT HUB. Spokes therefore learn only the hub&#x27;s routes, and spoke-to-spoke traffic passes through the hub." xmlns="http://www.w3.org/2000/svg">
<style>
.m7b{ font-family:var(--sans),sans-serif; }
.m7b .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.m7b .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.m7b .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.m7b .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.m7b .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.m7b .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.m7b .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.m7b .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.m7b .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.m7b .h{ fill:var(--text); font-size:15px; font-weight:700; }
.m7b .b{ fill:var(--text); font-size:13px; font-weight:700; }
.m7b .s{ fill:var(--text-dim); font-size:13px; }
.m7b .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.m7b .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.m7b .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.m7b .c{ fill:var(--text-faint); font-size:12.5px; }
.m7b .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.m7b .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.m7b .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.m7b .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m7b-a); }
.m7b .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m7b-a); marker-start:url(#m7b-s); }
.m7b .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#m7b-g); }
.m7b .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#m7b-a); }
.m7b .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.m7b .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.m7b .bad{ stroke:var(--text); stroke-width:3; fill:none; }
</style>
<defs><marker id="m7b-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="m7b-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="m7b-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker></defs>
<g class="m7b">
<rect x="280" y="20" width="160" height="70" rx="7" class="dark"/>
<text x="360.0" y="41.5" text-anchor="middle" class="on" style="fill:var(--bg)">hub VRF</text>
<text x="360.0" y="59.5" text-anchor="middle" class="on" style="fill:var(--bg)">import SPOKE</text>
<text x="360.0" y="77.5" text-anchor="middle" class="on" style="fill:var(--bg)">export HUB</text>
<rect x="40" y="180" width="160" height="70" rx="7" class="acc"/>
<text x="120.0" y="201.5" text-anchor="middle" class="b">spoke 1</text>
<text x="120.0" y="219.5" text-anchor="middle" class="m">import HUB</text>
<text x="120.0" y="237.5" text-anchor="middle" class="m">export SPOKE</text>
<line x1="120" y1="178" x2="360" y2="92" class="ar2"/>
<rect x="280" y="180" width="160" height="70" rx="7" class="acc"/>
<text x="360.0" y="201.5" text-anchor="middle" class="b">spoke 2</text>
<text x="360.0" y="219.5" text-anchor="middle" class="m">import HUB</text>
<text x="360.0" y="237.5" text-anchor="middle" class="m">export SPOKE</text>
<line x1="360" y1="178" x2="360" y2="92" class="ar2"/>
<rect x="520" y="180" width="160" height="70" rx="7" class="acc"/>
<text x="600.0" y="201.5" text-anchor="middle" class="b">spoke 3</text>
<text x="600.0" y="219.5" text-anchor="middle" class="m">import HUB</text>
<text x="600.0" y="237.5" text-anchor="middle" class="m">export SPOKE</text>
<line x1="600" y1="178" x2="360" y2="92" class="ar2"/>
<path d="M200 215 L278 215" class="gd"/>
<path d="M440 215 L518 215" class="gd"/>
<text x="240" y="205" text-anchor="middle" class="c">no route</text>
<text x="480" y="205" text-anchor="middle" class="c">no route</text>
<text x="360" y="278" text-anchor="middle" class="c">spoke to spoke goes via the hub</text>
</g>
</svg>
<figcaption>Asymmetric route targets build topologies. Here the hub sees everyone; the spokes see only the hub.</figcaption>
</figure>

Spoke VRFs export one RT and import another. The hub VRF does the reverse. Spokes learn only the hub's routes (often just a default route), so all spoke-to-spoke traffic passes through the hub site, typically where a central firewall sits. In practice the hub PE uses two VRFs, one facing in and one facing out through the hub CE, so traffic really does traverse the firewall.

### Extranets and shared services

<figure class="diagram">
<svg viewBox="0 0 720 250" style="width:100%;height:auto;max-width:720px" role="img" aria-label="An extranet: a SHARED services VRF exports RT 65000:99 and imports the RTs of RED and BLUE; RED and BLUE each import 65000:99 in addition to their own RT. Both customers reach the shared services, but not each other." xmlns="http://www.w3.org/2000/svg">
<style>
.m7c{ font-family:var(--sans),sans-serif; }
.m7c .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.m7c .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.m7c .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.m7c .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.m7c .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.m7c .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.m7c .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.m7c .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.m7c .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.m7c .h{ fill:var(--text); font-size:15px; font-weight:700; }
.m7c .b{ fill:var(--text); font-size:13px; font-weight:700; }
.m7c .s{ fill:var(--text-dim); font-size:13px; }
.m7c .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.m7c .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.m7c .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.m7c .c{ fill:var(--text-faint); font-size:12.5px; }
.m7c .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.m7c .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.m7c .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.m7c .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m7c-a); }
.m7c .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m7c-a); marker-start:url(#m7c-s); }
.m7c .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#m7c-g); }
.m7c .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#m7c-a); }
.m7c .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.m7c .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.m7c .bad{ stroke:var(--text); stroke-width:3; fill:none; }
</style>
<defs><marker id="m7c-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="m7c-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="m7c-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker></defs>
<g class="m7c">
<rect x="250" y="20" width="220" height="70" rx="7" class="dark"/>
<text x="360.0" y="41.5" text-anchor="middle" class="on" style="fill:var(--bg)">VRF SHARED (DNS, NTP)</text>
<text x="360.0" y="59.5" text-anchor="middle" class="on" style="fill:var(--bg)">export 65000:99</text>
<text x="360.0" y="77.5" text-anchor="middle" class="on" style="fill:var(--bg)">import 65000:10, 65000:20</text>
<rect x="40" y="160" width="240" height="64" rx="7" class="acc"/>
<text x="160.0" y="187.5" text-anchor="middle" class="b">VRF RED</text>
<text x="160.0" y="205.5" text-anchor="middle" class="m">import 65000:10, 65000:99</text>
<rect x="440" y="160" width="240" height="64" rx="7" class="pnl"/>
<text x="560.0" y="187.5" text-anchor="middle" class="b">VRF BLUE</text>
<text x="560.0" y="205.5" text-anchor="middle" class="m">import 65000:20, 65000:99</text>
<line x1="160" y1="158" x2="300" y2="92" class="ar2"/>
<line x1="560" y1="158" x2="420" y2="92" class="ar2"/>
<line x1="282" y1="192" x2="438" y2="192" class="gd"/>
<text x="360" y="182" text-anchor="middle" class="c">never mixed</text>
</g>
</svg>
<figcaption>Extranets and shared services are just extra import RTs. Overlapping addresses must not be imported into the same VRF.</figcaption>
</figure>

A provider often runs shared services, such as DNS, NTP, or a monitoring network, that every customer must reach. A **shared services VRF** imports the RTs of all customers and exports its own RT, which every customer VRF also imports. Customers reach the services, but not each other, because no customer VRF imports another customer's RT.

The same mechanism builds an **extranet** between two companies that want limited connectivity. There is one constraint: addresses imported into the same VRF must not overlap, so extranet partners must agree on non-overlapping ranges for the routes they share. Route maps on import and export can restrict exactly which prefixes cross.

## Connecting the customer

<figure class="diagram">
<svg viewBox="0 0 720 260" style="width:100%;height:auto;max-width:720px" role="img" aria-label="PE-CE routing options: static routes, the simplest; eBGP, the most common, with AS override or allowas-in when every site uses the same AS; and OSPF or IS-IS, where the provider network looks like part of the customer&#x27;s IGP." xmlns="http://www.w3.org/2000/svg">
<style>
.m7d{ font-family:var(--sans),sans-serif; }
.m7d .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.m7d .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.m7d .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.m7d .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.m7d .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.m7d .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.m7d .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.m7d .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.m7d .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.m7d .h{ fill:var(--text); font-size:15px; font-weight:700; }
.m7d .b{ fill:var(--text); font-size:13px; font-weight:700; }
.m7d .s{ fill:var(--text-dim); font-size:13px; }
.m7d .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.m7d .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.m7d .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.m7d .c{ fill:var(--text-faint); font-size:12.5px; }
.m7d .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.m7d .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.m7d .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.m7d .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m7d-a); }
.m7d .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m7d-a); marker-start:url(#m7d-s); }
.m7d .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#m7d-g); }
.m7d .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#m7d-a); }
.m7d .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.m7d .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.m7d .bad{ stroke:var(--text); stroke-width:3; fill:none; }
</style>
<defs><marker id="m7d-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="m7d-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="m7d-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker></defs>
<g class="m7d">
<rect x="20" y="30" width="216" height="66" rx="7" class="box"/>
<text x="128.0" y="58.5" text-anchor="middle" class="b">static</text>
<text x="128.0" y="76.5" text-anchor="middle" class="c">simplest; no dynamic failover</text>
<rect x="252" y="30" width="216" height="66" rx="7" class="acc"/>
<text x="360.0" y="58.5" text-anchor="middle" class="b">eBGP</text>
<text x="360.0" y="76.5" text-anchor="middle" class="s">most common; policy, scale</text>
<rect x="484" y="30" width="216" height="66" rx="7" class="pnl"/>
<text x="592.0" y="58.5" text-anchor="middle" class="b">OSPF / IS-IS</text>
<text x="592.0" y="76.5" text-anchor="middle" class="c">fits an existing customer IGP</text>
<rect x="20" y="130" width="680" height="96" rx="7" class="pnl"/>
<text x="360.0" y="161.5" text-anchor="middle" class="b">eBGP with one AS at every site</text>
<text x="360.0" y="182.5" text-anchor="middle" class="s">Site B rejects routes from site A because its own AS 65001 is already in the AS path.</text>
<text x="360.0" y="203.5" text-anchor="middle" class="s">Fix it with as-override on the PE, or allowas-in on the CE; use Site of Origin to stop loops.</text>
</g>
</svg>
<figcaption>The PE-CE protocol is the customer-facing half of the service. eBGP is the usual choice.</figcaption>
</figure>

The **PE-CE protocol** carries routes between the customer's router and the VRF.

- **Static routing** is the simplest and needs no protocol on the CE, but a failed link is not detected dynamically.
- **eBGP** is the most common choice. It scales, supports policy, and makes multihomed sites easy. Each site normally has a private ASN.
- **OSPF or IS-IS** suits customers whose sites already run one IGP. RFC 4577 defines how OSPF routes keep their type across the VPN, so the provider network appears to the customer as a special part of its OSPF domain.

A common surprise with eBGP: if every site of a customer uses the same ASN, site B rejects routes from site A, because its own ASN already appears in the AS path. Two fixes exist. **AS override** makes the PE replace the customer's ASN with the provider's when advertising to the CE. **allowas-in** on the CE accepts its own ASN. When sites are multihomed, either fix can create routing loops between sites; the **Site of Origin (SoO)** extended community marks routes with their site of origin so a PE never sends a route back to the site it came from.

## Internet access

Customers inside a VPN usually also want the Internet. The common designs are:

- **Through the customer's own hub.** The Internet connection lives at one customer site, and other sites follow a default route to it. This needs nothing special from the provider.
- **A separate Internet connection** on the same CE, often on a second VLAN to the PE that is not in any VRF. Simple, and keeps the VPN clean.
- **Route leaking.** The provider injects a default route into the VRF that points at an Internet gateway, and leaks the customer's public prefixes (or a NAT pool) back. This is flexible but needs care, because private addresses must be translated somewhere.

Many providers sell Internet access itself as just another VRF, with all Internet routes imported into an "Internet VRF" so that the global table stays small.

## Between providers: inter-AS options

When a customer has sites on two providers' networks, the providers must join their VPNs. RFC 4364 describes three options.

<figure class="diagram">
<svg viewBox="0 0 720 330" style="width:100%;height:auto;max-width:720px" role="img" aria-label="Inter-AS VPN options. Option A: the two ASBRs connect back to back with one VRF and one interface per VPN, exchanging plain IP routes. Option B: the ASBRs exchange VPNv4 routes over eBGP and swap VPN labels. Option C: the ASBRs exchange PE loopbacks with labels (BGP labeled unicast) and the PEs or route reflectors exchange VPNv4 routes directly, multihop." xmlns="http://www.w3.org/2000/svg">
<style>
.m7e{ font-family:var(--sans),sans-serif; }
.m7e .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.m7e .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.m7e .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.m7e .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.m7e .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.m7e .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.m7e .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.m7e .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.m7e .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.m7e .h{ fill:var(--text); font-size:15px; font-weight:700; }
.m7e .b{ fill:var(--text); font-size:13px; font-weight:700; }
.m7e .s{ fill:var(--text-dim); font-size:13px; }
.m7e .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.m7e .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.m7e .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.m7e .c{ fill:var(--text-faint); font-size:12.5px; }
.m7e .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.m7e .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.m7e .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.m7e .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m7e-a); }
.m7e .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m7e-a); marker-start:url(#m7e-s); }
.m7e .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#m7e-g); }
.m7e .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#m7e-a); }
.m7e .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.m7e .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.m7e .bad{ stroke:var(--text); stroke-width:3; fill:none; }
</style>
<defs><marker id="m7e-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="m7e-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="m7e-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker></defs>
<g class="m7e">
<rect x="20" y="30" width="120" height="60" rx="7" class="dark"/>
<text x="80.0" y="64.5" text-anchor="middle" class="on" style="fill:var(--bg)">Option A</text>
<rect x="150" y="30" width="350" height="60" rx="7" class="pnl"/>
<text x="325.0" y="64.5" text-anchor="middle" class="s">one VRF and VLAN per VPN at the border</text>
<rect x="510" y="30" width="200" height="60" rx="7" class="box"/>
<text x="610.0" y="64.5" text-anchor="middle" class="c">simple; much ASBR state</text>
<rect x="20" y="130" width="120" height="60" rx="7" class="dark"/>
<text x="80.0" y="164.5" text-anchor="middle" class="on" style="fill:var(--bg)">Option B</text>
<rect x="150" y="130" width="350" height="60" rx="7" class="pnl"/>
<text x="325.0" y="164.5" text-anchor="middle" class="s">eBGP VPNv4 between ASBRs, labels swapped</text>
<rect x="510" y="130" width="200" height="60" rx="7" class="box"/>
<text x="610.0" y="164.5" text-anchor="middle" class="c">no per-VPN interfaces</text>
<rect x="20" y="230" width="120" height="60" rx="7" class="dark"/>
<text x="80.0" y="264.5" text-anchor="middle" class="on" style="fill:var(--bg)">Option C</text>
<rect x="150" y="230" width="350" height="60" rx="7" class="pnl"/>
<text x="325.0" y="264.5" text-anchor="middle" class="s">BGP-LU loopbacks; VPNv4 between RRs</text>
<rect x="510" y="230" width="200" height="60" rx="7" class="box"/>
<text x="610.0" y="264.5" text-anchor="middle" class="c">most scalable; most trust</text>
</g>
</svg>
<figcaption>Three ways to join two providers' VPNs, from simplest to most scalable (RFC 4364 section 10).</figcaption>
</figure>

- **Option A, back-to-back VRFs.** The two border routers (ASBRs) connect with one subinterface per VPN, and each treats the other as a CE. Plain IP crosses the border. It is simple and keeps the providers isolated, but every VPN needs configuration and state on both ASBRs, so it scales poorly.
- **Option B.** The ASBRs exchange VPNv4 routes directly over eBGP and swap VPN labels at the border. No per-VPN interfaces are needed, but the ASBRs still hold every shared VPN route.
- **Option C.** The providers exchange only their PE loopbacks, with labels, using BGP labeled unicast (Chapter 5). VPNv4 routes then flow directly between route reflectors in the two networks over multihop eBGP. It scales best, but each provider must trust the other with its internal loopbacks and label space.

A related arrangement, **Carrier Supporting Carrier**, lets one provider carry another provider's labelled traffic, so a small carrier can use a large carrier as its MPLS core.

## Hands on: playing with route targets

Use the L3VPN lab from Chapter 6 (`labs/lab4-l3vpn`). The exercises edit configuration in `vtysh`; destroy and redeploy the lab to get back to the original state.

**Exercise 1: an accidental leak.** On pe2, make BLUE import RED's route target as well:

```text
pe2# configure terminal
pe2(config)# router bgp 65000 vrf BLUE
pe2(config-router)# address-family ipv4 unicast
pe2(config-router-af)# rt vpn import 65000:20 65000:10
pe2(config-router-af)# end
pe2# show bgp vrf BLUE ipv4 unicast 192.168.1.0/24
```

BLUE on pe2 now has two candidate paths for `192.168.1.0/24`: its own site behind pe1, and RED's. BGP picks one, and depending on which, traffic from ce4 may be delivered to RED's site. This is why extranets require non-overlapping addresses, and why RT changes in production go through careful review.

**Exercise 2: a shared services prefix.** Remove the extra import, then build a tiny shared service: add an address in a new range to ce2's loopback (`docker exec clab-lab4-l3vpn-ce2 ip addr add 10.99.0.1/32 dev lo`), add a static route for it in pe2's VRF RED, and export it with an extra RT using a route map on export. Import that RT into BLUE on pe1, and confirm that ce3 can reach `10.99.0.1` but nothing else of RED's. This exercise is open-ended; check your work with `show bgp ipv4 vpn` and the VRF routing tables.

## Summary

- **Route reflectors** remove the full iBGP mesh; **RT constraint** sends each PE only the routes it imports.
- Import and export **RTs** build topologies: **any-to-any**, **hub and spoke**, **extranets**, and **shared services**. Shared routes must not overlap.
- **eBGP** is the usual PE-CE protocol; **AS override**, **allowas-in**, and **Site of Origin** handle sites that share an ASN.
- **Internet access** is provided through a hub, a separate connection, or route leaking.
- **Inter-AS options A, B and C** join VPNs across providers, trading simplicity for scale.
