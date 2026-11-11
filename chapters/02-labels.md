# Chapter 2: Labels and Forwarding

This chapter looks at what a label actually is on the wire, what a router does when it receives one, and how the forwarding tables are organised. It ends with a lab in which you build a working LSP by hand, without any signalling protocol, so that every label decision is visible.

## The label stack entry

An MPLS label is carried in a 32-bit **label stack entry**.

<figure class="diagram">
<svg viewBox="0 0 720 200" style="width:100%;height:auto;max-width:720px" role="img" aria-label="A 32-bit label stack entry: a 20-bit label value, a 3-bit traffic class, a 1-bit bottom-of-stack flag, and an 8-bit time to live." xmlns="http://www.w3.org/2000/svg">
<style>
.m2a{ font-family:var(--sans),sans-serif; }
.m2a .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.m2a .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.m2a .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.m2a .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.m2a .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.m2a .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.m2a .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.m2a .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.m2a .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.m2a .h{ fill:var(--text); font-size:15px; font-weight:700; }
.m2a .b{ fill:var(--text); font-size:13px; font-weight:700; }
.m2a .s{ fill:var(--text-dim); font-size:13px; }
.m2a .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.m2a .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.m2a .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.m2a .c{ fill:var(--text-faint); font-size:12.5px; }
.m2a .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.m2a .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.m2a .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.m2a .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m2a-a); }
.m2a .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m2a-a); marker-start:url(#m2a-s); }
.m2a .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#m2a-g); }
.m2a .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#m2a-a); }
.m2a .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.m2a .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.m2a .bad{ stroke:var(--text); stroke-width:3; fill:none; }
</style>
<defs><marker id="m2a-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="m2a-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="m2a-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker></defs>
<g class="m2a">
<rect x="40" y="70" width="400" height="44" rx="3" class="acc"/>
<text x="240.0" y="96.5" text-anchor="middle" class="b">Label</text>
<text x="240.0" y="140" text-anchor="middle" class="c">20 bits</text>
<rect x="440" y="70" width="60" height="44" rx="3" class="pnl"/>
<text x="470.0" y="96.5" text-anchor="middle" class="s">TC</text>
<text x="470.0" y="140" text-anchor="middle" class="c">3 bits</text>
<rect x="500" y="70" width="20" height="44" rx="3" class="dark"/>
<text x="510.0" y="96.5" text-anchor="middle" class="on" style="fill:var(--bg)">S</text>
<text x="510.0" y="140" text-anchor="middle" class="c">1 bit</text>
<rect x="520" y="70" width="160" height="44" rx="3" class="pnl"/>
<text x="600.0" y="96.5" text-anchor="middle" class="s">TTL</text>
<text x="600.0" y="140" text-anchor="middle" class="c">8 bits</text>
<text x="40" y="58" text-anchor="middle" class="c">0</text>
<text x="440" y="58" text-anchor="middle" class="c">20</text>
<text x="500" y="58" text-anchor="middle" class="c">23</text>
<text x="520" y="58" text-anchor="middle" class="c">24</text>
<text x="680" y="58" text-anchor="middle" class="c">32</text>
<text x="240" y="30" text-anchor="middle" class="b">one label stack entry = 32 bits</text>
<text x="240" y="172" text-anchor="middle" class="s">value 0 to 1,048,575</text>
<text x="470.0" y="172" text-anchor="middle" class="s">QoS</text>
<text x="510.0" y="172" text-anchor="middle" class="s">last?</text>
<text x="600" y="172" text-anchor="middle" class="s">loop guard</text>
</g>
</svg>
<figcaption>The whole MPLS header. S is set to 1 only on the bottom label, so a router knows when the stack ends.</figcaption>
</figure>

- **Label (20 bits).** The value the next router looks up. About one million labels are possible per router.
- **TC (3 bits).** Traffic class, used for QoS. It was called EXP (experimental) in early documents, and you will still see that name.
- **S (1 bit).** Bottom of stack. It is 1 on the last label and 0 on every label above it.
- **TTL (8 bits).** Time to live, decremented at every hop exactly like the IP TTL, so a forwarding loop cannot circulate a packet forever.

A packet can carry several entries, one after another. This is the **label stack**, and stacking is what makes MPLS services work: the top label gets the packet across the core, and a label underneath tells the egress router what to do with it.

<figure class="diagram">
<svg viewBox="0 0 720 190" style="width:100%;height:auto;max-width:720px" role="img" aria-label="An Ethernet frame carrying MPLS: the Ethernet header with EtherType 0x8847, then a stack of two labels, the top label with S equal to 0 and the bottom label with S equal to 1, then the original IP packet." xmlns="http://www.w3.org/2000/svg">
<style>
.m2b{ font-family:var(--sans),sans-serif; }
.m2b .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.m2b .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.m2b .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.m2b .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.m2b .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.m2b .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.m2b .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.m2b .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.m2b .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.m2b .h{ fill:var(--text); font-size:15px; font-weight:700; }
.m2b .b{ fill:var(--text); font-size:13px; font-weight:700; }
.m2b .s{ fill:var(--text-dim); font-size:13px; }
.m2b .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.m2b .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.m2b .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.m2b .c{ fill:var(--text-faint); font-size:12.5px; }
.m2b .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.m2b .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.m2b .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.m2b .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m2b-a); }
.m2b .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m2b-a); marker-start:url(#m2b-s); }
.m2b .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#m2b-g); }
.m2b .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#m2b-a); }
.m2b .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.m2b .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.m2b .bad{ stroke:var(--text); stroke-width:3; fill:none; }
</style>
<defs><marker id="m2b-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="m2b-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="m2b-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker></defs>
<g class="m2b">
<rect x="20" y="60" width="150" height="34" rx="3" class="pnl"/>
<text x="95.0" y="81.5" text-anchor="middle" class="m">Ethernet</text>
<rect x="170" y="60" width="80" height="34" rx="3" class="hov"/>
<text x="210.0" y="81.5" text-anchor="middle" class="m">0x8847</text>
<rect x="250" y="60" width="140" height="34" rx="3" class="acc"/>
<text x="320.0" y="81.5" text-anchor="middle" class="m">label 24, S=0</text>
<rect x="390" y="60" width="140" height="34" rx="3" class="acc"/>
<text x="460.0" y="81.5" text-anchor="middle" class="m">label 17, S=1</text>
<rect x="530" y="60" width="150" height="34" rx="3" class="box"/>
<text x="605.0" y="81.5" text-anchor="middle" class="m">IP packet ...</text>
<text x="95" y="46" text-anchor="middle" class="c">dst MAC, src MAC</text>
<text x="210" y="46" text-anchor="middle" class="c">EtherType</text>
<path d="M250 112 L250 124 L530 124 L530 112" class="ln"/>
<text x="390" y="144" text-anchor="middle" class="s">label stack: as many entries as needed</text>
<text x="320" y="46" text-anchor="middle" class="a">top label</text>
<text x="460" y="46" text-anchor="middle" class="a">bottom label</text>
<text x="615" y="144" text-anchor="middle" class="c">untouched until the end</text>
<text x="360" y="176" text-anchor="middle" class="c">The label stack is a shim between layer 2 and layer 3, sometimes called layer 2.5.</text>
</g>
</svg>
<figcaption>Labels sit between the Ethernet header and the IP packet. EtherType 0x8847 tells the receiver to read a label, not an IP header.</figcaption>
</figure>

On Ethernet, a labelled packet is identified by **EtherType 0x8847**. A router reads labels until it finds one with S set, and only then expects the payload. The payload type is not written anywhere in the label; the egress router knows it from how the label was allocated.

## Push, swap and pop

Every MPLS forwarding decision is one of three operations on the top of the stack.

<figure class="diagram">
<svg viewBox="0 0 720 270" style="width:100%;height:auto;max-width:720px" role="img" aria-label="Along an LSP: the ingress router r1 pushes label 100 onto the IP packet, the transit router r2 swaps 100 for 200, and the egress router r3 pops 200 and forwards the plain IP packet to host h2." xmlns="http://www.w3.org/2000/svg">
<style>
.m2c{ font-family:var(--sans),sans-serif; }
.m2c .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.m2c .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.m2c .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.m2c .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.m2c .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.m2c .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.m2c .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.m2c .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.m2c .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.m2c .h{ fill:var(--text); font-size:15px; font-weight:700; }
.m2c .b{ fill:var(--text); font-size:13px; font-weight:700; }
.m2c .s{ fill:var(--text-dim); font-size:13px; }
.m2c .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.m2c .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.m2c .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.m2c .c{ fill:var(--text-faint); font-size:12.5px; }
.m2c .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.m2c .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.m2c .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.m2c .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m2c-a); }
.m2c .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m2c-a); marker-start:url(#m2c-s); }
.m2c .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#m2c-g); }
.m2c .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#m2c-a); }
.m2c .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.m2c .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.m2c .bad{ stroke:var(--text); stroke-width:3; fill:none; }
</style>
<defs><marker id="m2c-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="m2c-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="m2c-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker></defs>
<g class="m2c">
<circle cx="50" cy="60" r="24" class="pnl"/>
<text x="50" y="64.5" text-anchor="middle" class="b">h1</text>
<circle cx="190" cy="60" r="24" class="dark"/>
<text x="190" y="64.5" text-anchor="middle" class="on" style="fill:var(--bg)">r1</text>
<circle cx="360" cy="60" r="24" class="pnl"/>
<text x="360" y="64.5" text-anchor="middle" class="b">r2</text>
<circle cx="530" cy="60" r="24" class="dark"/>
<text x="530" y="64.5" text-anchor="middle" class="on" style="fill:var(--bg)">r3</text>
<circle cx="670" cy="60" r="24" class="pnl"/>
<text x="670" y="64.5" text-anchor="middle" class="b">h2</text>
<line x1="74" y1="60" x2="166" y2="60" class="lk"/>
<line x1="214" y1="60" x2="336" y2="60" class="lk"/>
<line x1="384" y1="60" x2="506" y2="60" class="lk"/>
<line x1="554" y1="60" x2="646" y2="60" class="lk"/>
<text x="190" y="108" text-anchor="middle" class="a">PUSH 100</text>
<text x="360" y="108" text-anchor="middle" class="a">SWAP 100 to 200</text>
<text x="530" y="108" text-anchor="middle" class="a">POP 200</text>
<rect x="65" y="150" width="60" height="28" rx="3" class="box"/>
<text x="95.0" y="168.5" text-anchor="middle" class="m">IP</text>
<rect x="220" y="150" width="46" height="28" rx="3" class="acc"/>
<text x="243.0" y="168.5" text-anchor="middle" class="m">100</text>
<rect x="266" y="150" width="60" height="28" rx="3" class="box"/>
<text x="296.0" y="168.5" text-anchor="middle" class="m">IP</text>
<rect x="390" y="150" width="46" height="28" rx="3" class="acc"/>
<text x="413.0" y="168.5" text-anchor="middle" class="m">200</text>
<rect x="436" y="150" width="60" height="28" rx="3" class="box"/>
<text x="466.0" y="168.5" text-anchor="middle" class="m">IP</text>
<rect x="560" y="150" width="60" height="28" rx="3" class="box"/>
<text x="590.0" y="168.5" text-anchor="middle" class="m">IP</text>
<text x="95" y="202" text-anchor="middle" class="c">h1 to r1</text>
<text x="273" y="202" text-anchor="middle" class="c">r1 to r2</text>
<text x="443" y="202" text-anchor="middle" class="c">r2 to r3</text>
<text x="590" y="202" text-anchor="middle" class="c">r3 to h2</text>
<text x="360" y="248" text-anchor="middle" class="s">Labels have local meaning: 100 only means something on the link from r1 to r2.</text>
</g>
</svg>
<figcaption>The three label operations. An LSP is simply the chain of these decisions, one per router.</figcaption>
</figure>

- **Push** adds one or more labels. It happens at the ingress LER, and also in the middle of the network when a packet enters another tunnel.
- **Swap** replaces the top label with another. This is what transit LSRs do.
- **Pop** removes the top label. The router then looks at whatever is underneath: another label, or the IP packet.

Labels are **locally significant**. Label 100 in the figure means "send to r3 as 200" only on r2, and only because r2 told r1 to use 100. Another router can use 100 for something completely different. This is why labels must be distributed, by hand or by a protocol, between each pair of neighbours.

## Forwarding tables

An MPLS router keeps two related tables.

<figure class="diagram">
<svg viewBox="0 0 720 250" style="width:100%;height:auto;max-width:720px" role="img" aria-label="Two tables in an MPLS router. The FIB maps an IP prefix to a next hop and, at the ingress, a label to push. The label forwarding table maps an incoming label to an outgoing label and next hop." xmlns="http://www.w3.org/2000/svg">
<style>
.m2d{ font-family:var(--sans),sans-serif; }
.m2d .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.m2d .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.m2d .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.m2d .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.m2d .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.m2d .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.m2d .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.m2d .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.m2d .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.m2d .h{ fill:var(--text); font-size:15px; font-weight:700; }
.m2d .b{ fill:var(--text); font-size:13px; font-weight:700; }
.m2d .s{ fill:var(--text-dim); font-size:13px; }
.m2d .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.m2d .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.m2d .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.m2d .c{ fill:var(--text-faint); font-size:12.5px; }
.m2d .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.m2d .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.m2d .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.m2d .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m2d-a); }
.m2d .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m2d-a); marker-start:url(#m2d-s); }
.m2d .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#m2d-g); }
.m2d .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#m2d-a); }
.m2d .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.m2d .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.m2d .bad{ stroke:var(--text); stroke-width:3; fill:none; }
</style>
<defs><marker id="m2d-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="m2d-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="m2d-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker></defs>
<g class="m2d">
<rect x="20" y="30" width="330" height="190" rx="7" class="pnl"/>
<text x="185" y="56" text-anchor="middle" class="b">FIB (IP lookup, at the ingress)</text>
<text x="40" y="92" text-anchor="start" class="c">prefix</text>
<text x="170" y="92" text-anchor="start" class="c">next hop</text>
<text x="290" y="92" text-anchor="start" class="c">push</text>
<text x="40" y="126" text-anchor="start" class="m">10.2.2.0/24</text>
<text x="170" y="126" text-anchor="start" class="m">10.1.12.2</text>
<text x="290" y="126" text-anchor="start" class="ma">100</text>
<text x="40" y="160" text-anchor="start" class="m">10.0.0.2/32</text>
<text x="170" y="160" text-anchor="start" class="m">10.10.1.2</text>
<text x="290" y="160" text-anchor="start" class="ma">16</text>
<rect x="370" y="30" width="330" height="190" rx="7" class="pnl"/>
<text x="535" y="56" text-anchor="middle" class="b">LFIB (label lookup, everywhere)</text>
<text x="390" y="92" text-anchor="start" class="c">in</text>
<text x="470" y="92" text-anchor="start" class="c">out</text>
<text x="560" y="92" text-anchor="start" class="c">next hop</text>
<text x="390" y="126" text-anchor="start" class="ma">100</text>
<text x="470" y="126" text-anchor="start" class="ma">200</text>
<text x="560" y="126" text-anchor="start" class="m">10.1.23.2</text>
<text x="390" y="160" text-anchor="start" class="ma">200</text>
<text x="470" y="160" text-anchor="start" class="ma">pop</text>
<text x="560" y="160" text-anchor="start" class="m">10.2.2.2</text>
<text x="185" y="200" text-anchor="middle" class="c">ip route / show ip route</text>
<text x="535" y="200" text-anchor="middle" class="c">ip -M route / show mpls table</text>
</g>
</svg>
<figcaption>The FIB decides which label a packet gets; the LFIB decides what happens to a labelled packet. A label binding connects the two.</figcaption>
</figure>

- The ordinary **FIB** maps IP prefixes to next hops. On an ingress LER, an entry can also say "push these labels". The prefix (or group of prefixes) is the FEC.
- The **label forwarding table**, often called the **LFIB**, maps an incoming label to an action: swap to a given label and send to a next hop, or pop and continue.

The RFCs use more formal names. The incoming-label map is the **ILM**, and each outgoing action is a **NHLFE** (next hop label forwarding entry). Linux and FRR simply call it the MPLS table.

## Penultimate hop popping

Consider the egress router. If the last label is still on the packet when it arrives, the egress must do two lookups: first the label, to discover that it should pop it, then the IP destination. To save that work, MPLS lets the egress ask its upstream neighbour to pop the label one hop early. This is **penultimate hop popping (PHP)**.

<figure class="diagram">
<svg viewBox="0 0 720 290" style="width:100%;height:auto;max-width:720px" role="img" aria-label="Penultimate hop popping: with implicit null, the router before the egress pops the label so the egress does a single IP lookup. With explicit null, label 0 is kept to the egress so the traffic class bits survive." xmlns="http://www.w3.org/2000/svg">
<style>
.m2e{ font-family:var(--sans),sans-serif; }
.m2e .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.m2e .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.m2e .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.m2e .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.m2e .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.m2e .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.m2e .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.m2e .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.m2e .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.m2e .h{ fill:var(--text); font-size:15px; font-weight:700; }
.m2e .b{ fill:var(--text); font-size:13px; font-weight:700; }
.m2e .s{ fill:var(--text-dim); font-size:13px; }
.m2e .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.m2e .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.m2e .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.m2e .c{ fill:var(--text-faint); font-size:12.5px; }
.m2e .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.m2e .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.m2e .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.m2e .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m2e-a); }
.m2e .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m2e-a); marker-start:url(#m2e-s); }
.m2e .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#m2e-g); }
.m2e .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#m2e-a); }
.m2e .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.m2e .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.m2e .bad{ stroke:var(--text); stroke-width:3; fill:none; }
</style>
<defs><marker id="m2e-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="m2e-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="m2e-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker></defs>
<g class="m2e">
<text x="20" y="26" text-anchor="start" class="b">implicit null (label 3): PHP, the default</text>
<circle cx="80" cy="66" r="24" class="dark"/>
<text x="80" y="70.5" text-anchor="middle" class="on" style="fill:var(--bg)">r1</text>
<circle cx="260" cy="66" r="24" class="pnl"/>
<text x="260" y="70.5" text-anchor="middle" class="b">r2</text>
<circle cx="440" cy="66" r="24" class="dark"/>
<text x="440" y="70.5" text-anchor="middle" class="on" style="fill:var(--bg)">r3</text>
<line x1="104" y1="66" x2="236" y2="66" class="lk"/>
<line x1="284" y1="66" x2="416" y2="66" class="lk"/>
<rect x="127" y="86" width="40" height="28" rx="3" class="acc"/>
<text x="147.0" y="104.5" text-anchor="middle" class="m">100</text>
<rect x="167" y="86" width="46" height="28" rx="3" class="box"/>
<text x="190.0" y="104.5" text-anchor="middle" class="m">IP</text>
<rect x="327" y="86" width="46" height="28" rx="3" class="box"/>
<text x="350.0" y="104.5" text-anchor="middle" class="m">IP</text>
<rect x="490" y="46" width="220" height="40" rx="7" class="pnl"/>
<text x="600.0" y="70.5" text-anchor="middle" class="s">egress does one IP lookup</text>
<text x="260" y="136" text-anchor="middle" class="c">r2 pops (it is the penultimate hop)</text>
<text x="20" y="166" text-anchor="start" class="b">explicit null (label 0): no PHP</text>
<circle cx="80" cy="206" r="24" class="dark"/>
<text x="80" y="210.5" text-anchor="middle" class="on" style="fill:var(--bg)">r1</text>
<circle cx="260" cy="206" r="24" class="pnl"/>
<text x="260" y="210.5" text-anchor="middle" class="b">r2</text>
<circle cx="440" cy="206" r="24" class="dark"/>
<text x="440" y="210.5" text-anchor="middle" class="on" style="fill:var(--bg)">r3</text>
<line x1="104" y1="206" x2="236" y2="206" class="lk"/>
<line x1="284" y1="206" x2="416" y2="206" class="lk"/>
<rect x="127" y="226" width="40" height="28" rx="3" class="acc"/>
<text x="147.0" y="244.5" text-anchor="middle" class="m">100</text>
<rect x="167" y="226" width="46" height="28" rx="3" class="box"/>
<text x="190.0" y="244.5" text-anchor="middle" class="m">IP</text>
<rect x="307" y="226" width="40" height="28" rx="3" class="acc"/>
<text x="327.0" y="244.5" text-anchor="middle" class="m">0</text>
<rect x="347" y="226" width="46" height="28" rx="3" class="box"/>
<text x="370.0" y="244.5" text-anchor="middle" class="m">IP</text>
<rect x="490" y="186" width="220" height="40" rx="7" class="acc"/>
<text x="600.0" y="210.5" text-anchor="middle" class="b">TC bits arrive at the egress</text>
<text x="260" y="276" text-anchor="middle" class="c">r2 swaps 100 to 0; r3 pops 0, then routes</text>
</g>
</svg>
<figcaption>Implicit null asks the upstream router to pop. Explicit null keeps a label on the last hop when QoS marking must reach the egress.</figcaption>
</figure>

The egress asks for PHP by advertising the reserved label **3, implicit null**. Implicit null never appears on the wire; it simply means "pop before sending to me". If the egress needs to see the label, for example because the TC bits carry QoS marking that must survive to the last hop, it advertises **0, IPv4 explicit null** instead. The upstream router then swaps to label 0, and the egress pops it.

PHP only removes the **top** label. When a packet carries a VPN label underneath, that label still reaches the egress PE, which is exactly what VPNs need.

## The label space

<figure class="diagram">
<svg viewBox="0 0 720 170" style="width:100%;height:auto;max-width:720px" role="img" aria-label="The 20-bit label space: labels 0 to 15 are reserved, labels from 16 upward are free for local use. In this book the Segment Routing global block is 16000 to 23999 and dynamic labels are allocated from 16 upward by the protocols." xmlns="http://www.w3.org/2000/svg">
<style>
.m2f{ font-family:var(--sans),sans-serif; }
.m2f .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.m2f .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.m2f .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.m2f .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.m2f .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.m2f .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.m2f .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.m2f .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.m2f .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.m2f .h{ fill:var(--text); font-size:15px; font-weight:700; }
.m2f .b{ fill:var(--text); font-size:13px; font-weight:700; }
.m2f .s{ fill:var(--text-dim); font-size:13px; }
.m2f .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.m2f .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.m2f .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.m2f .c{ fill:var(--text-faint); font-size:12.5px; }
.m2f .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.m2f .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.m2f .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.m2f .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m2f-a); }
.m2f .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m2f-a); marker-start:url(#m2f-s); }
.m2f .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#m2f-g); }
.m2f .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#m2f-a); }
.m2f .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.m2f .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.m2f .bad{ stroke:var(--text); stroke-width:3; fill:none; }
</style>
<defs><marker id="m2f-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="m2f-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="m2f-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker></defs>
<g class="m2f">
<rect x="30" y="50" width="60" height="44" rx="3" class="dark"/>
<text x="60.0" y="76.5" text-anchor="middle" class="on" style="fill:var(--bg)">0 to 15</text>
<rect x="90" y="50" width="200" height="44" rx="3" class="pnl"/>
<text x="190.0" y="76.5" text-anchor="middle" class="s">dynamic (LDP, BGP)</text>
<rect x="290" y="50" width="150" height="44" rx="3" class="acc"/>
<text x="365.0" y="76.5" text-anchor="middle" class="b">SRGB 16000+</text>
<rect x="440" y="50" width="250" height="44" rx="3" class="pnl"/>
<text x="565.0" y="76.5" text-anchor="middle" class="s">free, up to 1,048,575</text>
<text x="60" y="116" text-anchor="middle" class="c">reserved</text>
<text x="365" y="116" text-anchor="middle" class="c">Segment Routing</text>
<text x="360" y="148" text-anchor="middle" class="c">0 IPv4 explicit null, 2 IPv6 explicit null, 3 implicit null, 7 entropy label indicator, 13 GAL</text>
</g>
</svg>
<figcaption>How the label space is carved up. Each router owns its own space; a label value means nothing on another router.</figcaption>
</figure>

Labels 0 to 15 are reserved for special meanings. The rest belong to each router to allocate as it likes. Protocols such as LDP and BGP take labels from a dynamic pool starting at 16. Segment Routing (Chapter 9) sets aside a block of labels, the **SRGB**; in this book it is 16000 to 23999, a common choice.

| Label | Meaning |
|---|---|
| 0 | IPv4 explicit null |
| 1 | Router alert |
| 2 | IPv6 explicit null |
| 3 | Implicit null (advertised only, never sent) |
| 7 | Entropy label indicator (load balancing) |
| 13 | Generic associated channel label (OAM) |

## TTL and traceroute

At the ingress, the IP TTL is normally copied into the new label's TTL, and at the egress the label TTL is copied back. As a result, `traceroute` through an MPLS network shows every router, and many implementations also report the labels each hop saw. Operators sometimes disable TTL propagation to hide the core from customers; the whole provider network then appears as a single hop.

What happens when the TTL expires on a P router depends on the implementation. The MPLS architecture lets the P router build an ICMP "time exceeded" message and send it **onward along the LSP**, so the egress can route it back to the sender, which works even when the P router has no route to the sender at all. Many commercial routers do exactly this. The Linux kernel's MPLS code does not generate these messages, so in the labs a `traceroute` across the core may show asterisks for the P routers. That is a property of the lab platform, not of MPLS.

## MTU

Every label adds 4 bytes. A packet with two labels is 8 bytes larger than the IP packet it carries, so core links need an MTU at least that much larger than the customer's, or large packets will be dropped. Service providers typically run core links at 9000 bytes or more for this reason. In the container labs the veth links default to 1500 bytes, which is fine for `ping`, but keep it in mind for real networks.

## Hands on: an LSP built by hand

This lab builds one LSP from `r1` to `r3` with static labels. It uses FRR for configuration, but no signalling protocol runs: every label is typed in. Chapter 3 explains the lab environment in detail; if you have not set it up yet, read that chapter first and come back.

<figure class="diagram">
<svg viewBox="0 0 720 230" style="width:100%;height:auto;max-width:720px" role="img" aria-label="Lab 1 topology: host h1 at 10.1.1.2 connects to r1, then r2, then r3, then host h2 at 10.2.2.2. r1 pushes label 100, r2 swaps to 200, r3 pops. Return traffic from h2 to h1 is routed as plain IP." xmlns="http://www.w3.org/2000/svg">
<style>
.m2g{ font-family:var(--sans),sans-serif; }
.m2g .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.m2g .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.m2g .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.m2g .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.m2g .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.m2g .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.m2g .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.m2g .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.m2g .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.m2g .h{ fill:var(--text); font-size:15px; font-weight:700; }
.m2g .b{ fill:var(--text); font-size:13px; font-weight:700; }
.m2g .s{ fill:var(--text-dim); font-size:13px; }
.m2g .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.m2g .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.m2g .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.m2g .c{ fill:var(--text-faint); font-size:12.5px; }
.m2g .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.m2g .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.m2g .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.m2g .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m2g-a); }
.m2g .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m2g-a); marker-start:url(#m2g-s); }
.m2g .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#m2g-g); }
.m2g .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#m2g-a); }
.m2g .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.m2g .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.m2g .bad{ stroke:var(--text); stroke-width:3; fill:none; }
</style>
<defs><marker id="m2g-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="m2g-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="m2g-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker></defs>
<g class="m2g">
<circle cx="50" cy="80" r="24" class="pnl"/>
<text x="50" y="84.5" text-anchor="middle" class="b">h1</text>
<text x="50" y="121" text-anchor="middle" class="c">10.1.1.2</text>
<circle cx="200" cy="80" r="24" class="dark"/>
<text x="200" y="84.5" text-anchor="middle" class="on" style="fill:var(--bg)">r1</text>
<circle cx="360" cy="80" r="24" class="pnl"/>
<text x="360" y="84.5" text-anchor="middle" class="b">r2</text>
<circle cx="520" cy="80" r="24" class="dark"/>
<text x="520" y="84.5" text-anchor="middle" class="on" style="fill:var(--bg)">r3</text>
<circle cx="670" cy="80" r="24" class="pnl"/>
<text x="670" y="84.5" text-anchor="middle" class="b">h2</text>
<text x="670" y="121" text-anchor="middle" class="c">10.2.2.2</text>
<line x1="74" y1="80" x2="176" y2="80" class="lk"/>
<text x="125.0" y="66" text-anchor="middle" class="c">10.1.1.0/24</text>
<line x1="224" y1="80" x2="336" y2="80" class="lk"/>
<text x="280.0" y="66" text-anchor="middle" class="c">10.1.12.0/24</text>
<line x1="384" y1="80" x2="496" y2="80" class="lk"/>
<text x="440.0" y="66" text-anchor="middle" class="c">10.1.23.0/24</text>
<line x1="544" y1="80" x2="646" y2="80" class="lk"/>
<text x="595.0" y="66" text-anchor="middle" class="c">10.2.2.0/24</text>
<path d="M200 140 L520 140" class="ar"/>
<text x="360" y="162" text-anchor="middle" class="a">push 100, swap to 200, pop: an LSP toward 10.2.2.0/24</text>
<path d="M520 196 L200 196" class="ag"/>
<text x="360" y="218" text-anchor="middle" class="c">return traffic: ordinary IP routes, no labels</text>
</g>
</svg>
<figcaption>Lab 1. One LSP in one direction, built entirely by hand, so every label decision is yours.</figcaption>
</figure>

The lab files are in `labs/lab1-static-lsp/`. The relevant configuration on each router:

```text
! r1, the ingress LER: push label 100 for traffic to h2's subnet
ip route 10.2.2.0/24 10.1.12.2 label 100

! r2, a transit LSR: swap incoming 100 for 200 toward r3
mpls lsp 100 10.1.23.2 200

! r3, the egress LER: pop 200 and forward the IP packet to h2
mpls lsp 200 10.2.2.2 implicit-null
```

The first line is a normal static route with a label to push. `mpls lsp IN NEXTHOP OUT` creates a label forwarding entry; using `implicit-null` as the outgoing label means pop. Deploy the lab and check each router:

```bash
cd labs/lab1-static-lsp
sudo containerlab deploy -t lab1-static-lsp.clab.yml

docker exec clab-lab1-static-lsp-r1 vtysh -c "show ip route 10.2.2.0/24"
docker exec clab-lab1-static-lsp-r2 vtysh -c "show mpls table"
docker exec clab-lab1-static-lsp-r3 vtysh -c "show mpls table"
```

On r2 you should see inbound label 100 with outbound label 200. The Linux kernel holds the same information. containerlab exposes each router's network namespace to the host under the container's name, so you can read it with the host's own tools:

```bash
sudo ip netns exec clab-lab1-static-lsp-r1 ip route show 10.2.2.0/24   # ... encap mpls 100 ...
sudo ip netns exec clab-lab1-static-lsp-r2 ip -f mpls route show       # 100 as to 200 via inet 10.1.23.2
```

Now send traffic from h1 to h2 and watch the labels on the wire. In one terminal, capture on r2; in another, ping:

```bash
sudo ip netns exec clab-lab1-static-lsp-r2 tcpdump -nni eth1 mpls      # label 100 arrives
sudo ip netns exec clab-lab1-static-lsp-r2 tcpdump -nni eth2 mpls      # label 200 leaves
docker exec clab-lab1-static-lsp-h1 ping -c 3 10.2.2.2
```

The replies from h2 travel back unlabelled, because no LSP exists in that direction. Two experiments are worth doing:

1. Remove the `mpls lsp` entry on r2 (`no mpls lsp 100 10.1.23.2 200` in `vtysh` configuration mode). The ping fails: r2 receives label 100 and has no idea what to do with it, so it drops the packet. An LSR never falls back to IP routing for a labelled packet.
2. Disable MPLS input on r2's interface: `docker exec clab-lab1-static-lsp-r2 sysctl -w net.mpls.conf.eth1.input=0`. The kernel now discards every labelled packet arriving on eth1. This setting is the most common reason a Linux MPLS lab "does nothing".

Destroy the lab with `sudo containerlab destroy -t lab1-static-lsp.clab.yml --cleanup`.

Typing labels by hand works for three routers and one destination. It does not scale, and nothing repairs it when the topology changes. That is the job of the label distribution protocols in Part II.

## Summary

- A label stack entry is 32 bits: a **20-bit label**, **3-bit TC**, **bottom-of-stack** bit, and **8-bit TTL**. Packets can carry a **stack** of them.
- Routers **push**, **swap**, or **pop** the top label. Labels are **locally significant**.
- The **FIB** maps prefixes to labels to push; the **LFIB** maps incoming labels to actions.
- **PHP** pops the last label one hop early (implicit null, 3); **explicit null** (0) keeps it to the egress.
- Every label adds 4 bytes; plan core **MTU** accordingly.
- In Linux, each interface must have **`net.mpls.conf.<if>.input=1`**, and **`net.mpls.platform_labels`** must be large enough.
