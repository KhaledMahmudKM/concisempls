# Chapter 3: The Lab

Every lab in this book runs on a single machine. The routers are **FRRouting** containers started by **containerlab**, and they forward labelled packets with the MPLS support built into the Linux kernel. This chapter explains how the pieces fit together, sets up the environment, and introduces the network you will use from Chapter 4 to the end.

## How the lab works

<figure class="diagram">
<svg viewBox="0 0 720 270" style="width:100%;height:auto;max-width:720px" role="img" aria-label="The lab stack: a Linux host kernel with the mpls_router module loaded, Docker on top, and containerlab starting FRR containers. Every container has its own network namespace and its own MPLS label table, but all share the host kernel." xmlns="http://www.w3.org/2000/svg">
<style>
.m3a{ font-family:var(--sans),sans-serif; }
.m3a .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.m3a .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.m3a .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.m3a .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.m3a .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.m3a .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.m3a .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.m3a .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.m3a .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.m3a .h{ fill:var(--text); font-size:15px; font-weight:700; }
.m3a .b{ fill:var(--text); font-size:13px; font-weight:700; }
.m3a .s{ fill:var(--text-dim); font-size:13px; }
.m3a .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.m3a .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.m3a .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.m3a .c{ fill:var(--text-faint); font-size:12.5px; }
.m3a .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.m3a .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.m3a .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.m3a .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m3a-a); }
.m3a .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m3a-a); marker-start:url(#m3a-s); }
.m3a .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#m3a-g); }
.m3a .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#m3a-a); }
.m3a .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.m3a .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.m3a .bad{ stroke:var(--text); stroke-width:3; fill:none; }
</style>
<defs><marker id="m3a-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="m3a-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="m3a-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker></defs>
<g class="m3a">
<rect x="20" y="210" width="680" height="44" rx="7" class="dark"/>
<text x="360.0" y="236.5" text-anchor="middle" class="on" style="fill:var(--bg)">host Linux kernel: mpls_router, mpls_iptunnel, vrf</text>
<rect x="20" y="160" width="680" height="40" rx="7" class="pnl"/>
<text x="360.0" y="184.5" text-anchor="middle" class="b">Docker engine  +  containerlab (reads the .clab.yml)</text>
<rect x="20" y="24" width="120" height="124" rx="7" class="acc"/>
<text x="80" y="46" text-anchor="middle" class="b">pe1</text>
<rect x="30" y="58" width="100" height="30" rx="5" class="pnl"/>
<text x="80.0" y="77.5" text-anchor="middle" class="s">FRR</text>
<rect x="30" y="98" width="100" height="38" rx="5" class="box"/>
<text x="80.0" y="112.5" text-anchor="middle" class="m">netns</text>
<text x="80.0" y="130.5" text-anchor="middle" class="c">label table</text>
<rect x="160" y="24" width="120" height="124" rx="7" class="acc"/>
<text x="220" y="46" text-anchor="middle" class="b">p1</text>
<rect x="170" y="58" width="100" height="30" rx="5" class="pnl"/>
<text x="220.0" y="77.5" text-anchor="middle" class="s">FRR</text>
<rect x="170" y="98" width="100" height="38" rx="5" class="box"/>
<text x="220.0" y="112.5" text-anchor="middle" class="m">netns</text>
<text x="220.0" y="130.5" text-anchor="middle" class="c">label table</text>
<rect x="300" y="24" width="120" height="124" rx="7" class="acc"/>
<text x="360" y="46" text-anchor="middle" class="b">p2</text>
<rect x="310" y="58" width="100" height="30" rx="5" class="pnl"/>
<text x="360.0" y="77.5" text-anchor="middle" class="s">FRR</text>
<rect x="310" y="98" width="100" height="38" rx="5" class="box"/>
<text x="360.0" y="112.5" text-anchor="middle" class="m">netns</text>
<text x="360.0" y="130.5" text-anchor="middle" class="c">label table</text>
<rect x="440" y="24" width="120" height="124" rx="7" class="acc"/>
<text x="500" y="46" text-anchor="middle" class="b">p3</text>
<rect x="450" y="58" width="100" height="30" rx="5" class="pnl"/>
<text x="500.0" y="77.5" text-anchor="middle" class="s">FRR</text>
<rect x="450" y="98" width="100" height="38" rx="5" class="box"/>
<text x="500.0" y="112.5" text-anchor="middle" class="m">netns</text>
<text x="500.0" y="130.5" text-anchor="middle" class="c">label table</text>
<rect x="580" y="24" width="120" height="124" rx="7" class="acc"/>
<text x="640" y="46" text-anchor="middle" class="b">pe2</text>
<rect x="590" y="58" width="100" height="30" rx="5" class="pnl"/>
<text x="640.0" y="77.5" text-anchor="middle" class="s">FRR</text>
<rect x="590" y="98" width="100" height="38" rx="5" class="box"/>
<text x="640.0" y="112.5" text-anchor="middle" class="m">netns</text>
<text x="640.0" y="130.5" text-anchor="middle" class="c">label table</text>
</g>
</svg>
<figcaption>Each router is a container. MPLS forwarding happens in the shared host kernel, which is why the module must be loaded on the host.</figcaption>
</figure>

- **FRR** is an open-source routing suite with IS-IS, OSPF, BGP, LDP, Segment Routing, and more. It is the same routing stack used inside SONiC and many commercial products. FRR computes routes and labels, and **zebra**, its kernel interface, installs them into Linux.
- **The Linux kernel** does the actual forwarding: IP routes, MPLS label operations, VRFs, and SRv6. MPLS forwarding has been in the kernel since version 4.1.
- **containerlab** reads a YAML topology file, starts one container per router, and wires them together with virtual Ethernet links. It also runs a few setup commands in each container after the links exist.

Each container is a separate network namespace with its own interfaces, routing tables, and MPLS label table. All of them share the host's kernel, which leads to the one requirement that catches most people out: **the MPLS kernel modules must be loaded on the host**. A container cannot load kernel modules for itself.

## Requirements

- A Linux machine, VM, or WSL2 distribution with Docker installed. Four CPU cores and 8 GB of RAM are plenty; the largest lab has eleven small containers.
- **containerlab**, installed with its one-line installer (`bash -c "$(curl -sL https://get.containerlab.dev)"`) or from its documentation.
- The MPLS modules, loaded on the host:

```bash
sudo modprobe mpls_router
sudo modprobe mpls_iptunnel
ls /proc/sys/net/mpls/          # should list platform_labels and conf/
```

If `ls` fails, your kernel was built without MPLS support. Most distribution kernels (Ubuntu, Debian, Fedora) include it as modules. If you are on WSL2 and the modules are missing, the simplest fix is a Linux VM; building a custom WSL kernel with `CONFIG_MPLS_ROUTING` and `CONFIG_MPLS_IPTUNNEL` also works.

- For the SRv6 lab in Chapter 12, a kernel of **6.1 or newer** (`uname -r`).

Pull the images once:

```bash
docker pull quay.io/frrouting/frr:10.7.0
docker pull alpine:3.20
```

The labs pin FRR 10.7.0, current as this book is written. Any 10.x release should behave the same; the SRv6 lab benefits most from a recent version.

## Kernel settings for MPLS

Two kinds of setting turn MPLS forwarding on inside each router's namespace:

| Setting | Meaning |
|---|---|
| `net.mpls.platform_labels` | Size of the label table. The default is 0, which disables MPLS entirely. The labs use 1048575, the maximum. |
| `net.mpls.conf.<interface>.input` | Accept labelled packets on this interface. Default 0, so every core-facing interface needs it set to 1. |

The topology files set both, so you rarely have to think about them, but they are the first thing to check when labelled packets disappear.

## Anatomy of a lab

Each lab lives in its own folder.

<figure class="diagram">
<svg viewBox="0 0 720 270" style="width:100%;height:auto;max-width:720px" role="img" aria-label="The labs folder: one folder per lab, each with a containerlab topology file and a configs folder holding a shared daemons file and one frr.conf per router." xmlns="http://www.w3.org/2000/svg">
<style>
.m3d{ font-family:var(--sans),sans-serif; }
.m3d .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.m3d .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.m3d .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.m3d .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.m3d .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.m3d .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.m3d .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.m3d .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.m3d .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.m3d .h{ fill:var(--text); font-size:15px; font-weight:700; }
.m3d .b{ fill:var(--text); font-size:13px; font-weight:700; }
.m3d .s{ fill:var(--text-dim); font-size:13px; }
.m3d .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.m3d .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.m3d .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.m3d .c{ fill:var(--text-faint); font-size:12.5px; }
.m3d .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.m3d .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.m3d .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.m3d .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m3d-a); }
.m3d .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m3d-a); marker-start:url(#m3d-s); }
.m3d .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#m3d-g); }
.m3d .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#m3d-a); }
.m3d .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.m3d .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.m3d .bad{ stroke:var(--text); stroke-width:3; fill:none; }
</style>
<defs><marker id="m3d-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="m3d-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="m3d-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker></defs>
<g class="m3d">
<text x="40" y="40" text-anchor="start" class="b">labs/</text>
<text x="66" y="68" text-anchor="start" class="m">README.md</text>
<text x="66" y="96" text-anchor="start" class="b">lab4-l3vpn/</text>
<text x="92" y="124" text-anchor="start" class="ma">lab4-l3vpn.clab.yml</text>
<text x="92" y="152" text-anchor="start" class="b">configs/</text>
<text x="118" y="180" text-anchor="start" class="m">daemons</text>
<text x="118" y="208" text-anchor="start" class="m">pe1/frr.conf</text>
<text x="118" y="236" text-anchor="start" class="m">p1/frr.conf  ...</text>
<rect x="380" y="30" width="320" height="56" rx="7" class="acc"/>
<text x="540.0" y="53.5" text-anchor="middle" class="s">topology: nodes, links,</text>
<text x="540.0" y="71.5" text-anchor="middle" class="s">sysctls and exec commands</text>
<rect x="380" y="100" width="320" height="56" rx="7" class="pnl"/>
<text x="540.0" y="123.5" text-anchor="middle" class="s">which FRR daemons start</text>
<text x="540.0" y="141.5" text-anchor="middle" class="c">(zebra always; isisd, ldpd, bgpd...)</text>
<rect x="380" y="170" width="320" height="56" rx="7" class="pnl"/>
<text x="540.0" y="193.5" text-anchor="middle" class="s">complete router configuration,</text>
<text x="540.0" y="211.5" text-anchor="middle" class="c">loaded by FRR at start</text>
<line x1="246" y1="120" x2="378" y2="58" class="ag"/>
<line x1="182" y1="176" x2="378" y2="128" class="ag"/>
<line x1="220" y1="204" x2="378" y2="198" class="ag"/>
</g>
</svg>
<figcaption>Every lab is plain text. The same frr.conf files work in containerlab, in your own Docker tooling, or on a VM.</figcaption>
</figure>

Here is the start of a topology file, from the L3VPN lab:

```yaml
name: lab4-l3vpn

topology:
  defaults:
    sysctls:
      net.mpls.platform_labels: 1048575
      net.ipv4.ip_forward: 1
  nodes:
    pe1:
      kind: linux
      image: quay.io/frrouting/frr:10.7.0
      binds:
        - configs/pe1/frr.conf:/etc/frr/frr.conf
        - configs/daemons:/etc/frr/daemons
      exec:
        - "ip link add RED type vrf table 10"
        - "ip link set eth3 master RED"
        - "sysctl -w net.mpls.conf.eth1.input=1"
        - "sysctl -w net.mpls.conf.eth2.input=1"
  links:
    - endpoints: ["pe1:eth1", "p1:eth1"]
```

- **`sysctls`** under `defaults` applies kernel settings to every container when it is created.
- **`binds`** mounts the router's configuration and the list of daemons into the container. FRR reads them when it starts.
- **`exec`** runs commands after the links exist: enabling MPLS input on each interface, and, in the VPN labs, creating VRFs.
- **`links`** wires interfaces together. `eth0` in every container is reserved for management; the lab interfaces start at `eth1`.

The daemons file switches FRR's protocol daemons on and off:

```text
bgpd=yes
isisd=yes
ldpd=yes
pathd=no
```

## Working with a lab

```bash
cd labs/lab2-ldp
sudo containerlab deploy -t lab2-ldp.clab.yml       # start
sudo containerlab inspect -t lab2-ldp.clab.yml      # list nodes and addresses
docker exec -it clab-lab2-ldp-pe1 vtysh             # FRR shell on pe1
docker exec -it clab-lab2-ldp-pe1 bash              # Linux shell on pe1
sudo containerlab destroy -t lab2-ldp.clab.yml --cleanup
```

Container names follow the pattern `clab-<lab>-<node>`. Inside `vtysh`, the commands look like those of most commercial routers: `show ip route`, `show isis neighbor`, `show mpls table`, `configure terminal`. Changes made in `vtysh` are lost when the lab is destroyed, unless you copy them back into the `frr.conf` files.

containerlab also makes each container's network namespace visible to the host under the container's name. That lets you use the host's own tools inside any router, which is handy because the router images are minimal:

```bash
sudo ip netns exec clab-lab2-ldp-p1 tcpdump -nni eth1        # capture on p1's eth1
sudo ip netns exec clab-lab2-ldp-p1 ip -f mpls route show    # p1's kernel label table
```

To look at packets in Wireshark, write the capture to a file with `tcpdump -w`, or pipe it straight in. Wireshark decodes MPLS, LDP, IS-IS, and BGP natively.

## The core network

From Chapter 4 onward, almost every lab uses the same five-router core.

<figure class="diagram">
<svg viewBox="0 0 720 330" style="width:100%;height:auto;max-width:720px" role="img" aria-label="The core topology used from Chapter 4 onward: PE1, P1, P2 and PE2 in a top row joined by links of metric 10; P3 below joined to PE1 with metric 10, to P1 with metric 10, and to PE2 with metric 25. Loopbacks are 10.0.0.1 for PE1, 10.0.0.11 to 10.0.0.13 for P1 to P3, and 10.0.0.2 for PE2." xmlns="http://www.w3.org/2000/svg">
<style>
.m3b{ font-family:var(--sans),sans-serif; }
.m3b .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.m3b .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.m3b .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.m3b .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.m3b .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.m3b .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.m3b .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.m3b .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.m3b .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.m3b .h{ fill:var(--text); font-size:15px; font-weight:700; }
.m3b .b{ fill:var(--text); font-size:13px; font-weight:700; }
.m3b .s{ fill:var(--text-dim); font-size:13px; }
.m3b .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.m3b .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.m3b .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.m3b .c{ fill:var(--text-faint); font-size:12.5px; }
.m3b .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.m3b .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.m3b .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.m3b .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m3b-a); }
.m3b .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m3b-a); marker-start:url(#m3b-s); }
.m3b .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#m3b-g); }
.m3b .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#m3b-a); }
.m3b .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.m3b .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.m3b .bad{ stroke:var(--text); stroke-width:3; fill:none; }
</style>
<defs><marker id="m3b-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="m3b-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="m3b-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker></defs>
<g class="m3b">
<line x1="118.0" y1="90.0" x2="242.0" y2="90.0" class="lk"/>
<line x1="298.0" y1="90.0" x2="422.0" y2="90.0" class="lk"/>
<line x1="478.0" y1="90.0" x2="602.0" y2="90.0" class="lk"/>
<line x1="114.1" y1="104.3" x2="335.9" y2="235.7" class="lk"/>
<line x1="384.1" y1="235.7" x2="605.9" y2="104.3" class="lk"/>
<line x1="283.7" y1="114.4" x2="346.3" y2="225.6" class="lk"/>
<circle cx="90" cy="90" r="28" class="dark"/>
<text x="90" y="94.5" text-anchor="middle" class="on" style="fill:var(--bg)">PE1</text>
<circle cx="270" cy="90" r="28" class="pnl"/>
<text x="270" y="94.5" text-anchor="middle" class="b">P1</text>
<circle cx="450" cy="90" r="28" class="pnl"/>
<text x="450" y="94.5" text-anchor="middle" class="b">P2</text>
<circle cx="630" cy="90" r="28" class="dark"/>
<text x="630" y="94.5" text-anchor="middle" class="on" style="fill:var(--bg)">PE2</text>
<circle cx="360" cy="250" r="28" class="pnl"/>
<text x="360" y="254.5" text-anchor="middle" class="b">P3</text>
<text x="90" y="48" text-anchor="middle" class="ma">10.0.0.1</text>
<text x="270" y="48" text-anchor="middle" class="ma">10.0.0.11</text>
<text x="450" y="48" text-anchor="middle" class="ma">10.0.0.12</text>
<text x="630" y="48" text-anchor="middle" class="ma">10.0.0.2</text>
<text x="360" y="305" text-anchor="middle" class="ma">10.0.0.13</text>
<text x="180" y="80" text-anchor="middle" class="c">10.10.1.0/24 (10)</text>
<text x="360" y="80" text-anchor="middle" class="c">10.10.2.0/24 (10)</text>
<text x="540" y="80" text-anchor="middle" class="c">10.10.3.0/24 (10)</text>
<text x="150" y="200" text-anchor="middle" class="c">10.10.4.0/24 (10)</text>
<text x="398" y="170" text-anchor="middle" class="c">10.10.6.0/24 (10)</text>
<text x="590" y="220" text-anchor="middle" class="a">10.10.5.0/24 (25)</text>
<text x="20" y="316" text-anchor="start" class="c">link metric in brackets</text>
</g>
</svg>
<figcaption>The lab core. Shortest path PE1 to PE2 is the top row (cost 30); the bottom path through P3 costs 35, which later chapters exploit.</figcaption>
</figure>

| Router | Role | Loopback | IS-IS NET |
|---|---|---|---|
| pe1 | Provider edge | 10.0.0.1 | 49.0001.0000.0000.0001.00 |
| pe2 | Provider edge | 10.0.0.2 | 49.0001.0000.0000.0002.00 |
| p1 | Core | 10.0.0.11 | 49.0001.0000.0000.0011.00 |
| p2 | Core | 10.0.0.12 | 49.0001.0000.0000.0012.00 |
| p3 | Core | 10.0.0.13 | 49.0001.0000.0000.0013.00 |

The IGP is **IS-IS**, level 2 only, with all links point-to-point. IS-IS is the usual choice in service provider cores and has the most complete Segment Routing support in FRR. Every core link is `10.10.N.0/24`, where the lower router name in the link gets `.1`. The link from p3 to pe2 has metric 25; all others have metric 10. That makes the shortest path from pe1 to pe2 run along the top (pe1, p1, p2, pe2: cost 30), with the path through p3 slightly longer (cost 35). Later chapters use that second path for fast reroute and traffic engineering.

The interface on each core router is configured like this:

```text
interface eth1
 description to p1
 ip address 10.10.1.1/24
 ip router isis CORE
 isis network point-to-point
exit
!
router isis CORE
 net 49.0001.0000.0000.0001.00
 is-type level-2-only
 metric-style wide
exit
```

## Customer sites

The service labs attach customer sites to the two PEs.

<figure class="diagram">
<svg viewBox="0 0 720 250" style="width:100%;height:auto;max-width:720px" role="img" aria-label="Customer sites attach to the PEs: on PE1, eth3 leads to CE1 and eth4 to CE3; on PE2, eth3 leads to CE2 and eth4 to CE4. CE1 and CE3 both use 192.168.1.0/24 and CE2 and CE4 both use 192.168.2.0/24, on purpose, to show overlapping customer addresses." xmlns="http://www.w3.org/2000/svg">
<style>
.m3c{ font-family:var(--sans),sans-serif; }
.m3c .box{ fill:var(--bg-soft); stroke:var(--border); stroke-width:1.5; }
.m3c .pnl{ fill:var(--panel); stroke:var(--border); stroke-width:1.5; }
.m3c .hov{ fill:var(--panel-hover); stroke:var(--border); stroke-width:1.5; }
.m3c .acc{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.6; }
.m3c .dark{ fill:var(--accent); stroke:var(--accent); stroke-width:1.6; }
.m3c .out{ fill:none; stroke:var(--accent); stroke-width:2; }
.m3c .dash{ fill:none; stroke:var(--accent); stroke-width:1.6; stroke-dasharray:6 4; }
.m3c .zone{ fill:none; stroke:var(--border); stroke-width:1.5; stroke-dasharray:6 4; }
.m3c .tint{ fill:var(--accent-dim); stroke:var(--accent); stroke-width:1.2; opacity:0.28; }
.m3c .h{ fill:var(--text); font-size:15px; font-weight:700; }
.m3c .b{ fill:var(--text); font-size:13px; font-weight:700; }
.m3c .s{ fill:var(--text-dim); font-size:13px; }
.m3c .a{ fill:var(--accent); font-size:13px; font-weight:700; }
.m3c .m{ fill:var(--text); font-size:12.5px; font-family:var(--mono),monospace; }
.m3c .ma{ fill:var(--accent); font-size:12.5px; font-family:var(--mono),monospace; }
.m3c .c{ fill:var(--text-faint); font-size:12.5px; }
.m3c .on{ fill:var(--bg); font-size:13px; font-weight:700; }
.m3c .ln{ stroke:var(--border); stroke-width:1.5; fill:none; }
.m3c .lk{ stroke:var(--text-faint); stroke-width:2.2; fill:none; }
.m3c .ar{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m3c-a); }
.m3c .ar2{ stroke:var(--accent); stroke-width:2; fill:none; marker-end:url(#m3c-a); marker-start:url(#m3c-s); }
.m3c .ag{ stroke:var(--text-faint); stroke-width:1.8; fill:none; marker-end:url(#m3c-g); }
.m3c .ad{ stroke:var(--accent); stroke-width:1.8; fill:none; stroke-dasharray:6 4; marker-end:url(#m3c-a); }
.m3c .gd{ stroke:var(--text-faint); stroke-width:1.6; fill:none; stroke-dasharray:4 4; }
.m3c .x{ stroke:var(--text-faint); stroke-width:2.4; fill:none; }
.m3c .bad{ stroke:var(--text); stroke-width:3; fill:none; }
</style>
<defs><marker id="m3c-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--accent)"/></marker><marker id="m3c-s" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M10 0 L0 5 L10 10 z" fill="var(--accent)"/></marker><marker id="m3c-g" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0 L10 5 L0 10 z" fill="var(--text-faint)"/></marker></defs>
<g class="m3c">
<rect x="230" y="60" width="260" height="130" rx="7" class="zone"/>
<text x="360" y="82" text-anchor="middle" class="c">core (from the previous figure)</text>
<circle cx="260" cy="125" r="24" class="dark"/>
<text x="260" y="129.5" text-anchor="middle" class="on" style="fill:var(--bg)">PE1</text>
<circle cx="460" cy="125" r="24" class="dark"/>
<text x="460" y="129.5" text-anchor="middle" class="on" style="fill:var(--bg)">PE2</text>
<line x1="284" y1="125" x2="436" y2="125" class="gd"/>
<circle cx="70" cy="70" r="24" class="acc"/>
<text x="70" y="74.5" text-anchor="middle" class="b">CE1</text>
<line x1="94" y1="70" x2="238" y2="117" class="lk"/>
<text x="140" y="60" text-anchor="middle" class="c">eth3</text>
<text x="70" y="36" text-anchor="middle" class="ma">192.168.1.1</text>
<circle cx="70" cy="190" r="24" class="pnl"/>
<text x="70" y="194.5" text-anchor="middle" class="b">CE3</text>
<line x1="94" y1="190" x2="238" y2="133" class="lk"/>
<text x="140" y="212" text-anchor="middle" class="c">eth4</text>
<text x="70" y="232" text-anchor="middle" class="ma">192.168.1.1</text>
<circle cx="650" cy="70" r="24" class="acc"/>
<text x="650" y="74.5" text-anchor="middle" class="b">CE2</text>
<line x1="626" y1="70" x2="482" y2="117" class="lk"/>
<text x="580" y="60" text-anchor="middle" class="c">eth3</text>
<text x="650" y="36" text-anchor="middle" class="ma">192.168.2.1</text>
<circle cx="650" cy="190" r="24" class="pnl"/>
<text x="650" y="194.5" text-anchor="middle" class="b">CE4</text>
<line x1="626" y1="190" x2="482" y2="133" class="lk"/>
<text x="580" y="212" text-anchor="middle" class="c">eth4</text>
<text x="650" y="232" text-anchor="middle" class="ma">192.168.2.1</text>
<text x="70" y="130" text-anchor="middle" class="s">site A</text>
<text x="650" y="130" text-anchor="middle" class="s">site B</text>
</g>
</svg>
<figcaption>Two customers, the same addresses. CE1 and CE2 belong to RED, CE3 and CE4 to BLUE; Chapter 6 keeps them apart.</figcaption>
</figure>

The CE routers are small Alpine Linux containers with a default route toward their PE and a "LAN" address on their loopback. The PEs reach them with static routes. RED and BLUE deliberately use **the same addresses**; separating them is the whole point of the VPN chapters.

## SONiC and other platforms

SONiC uses FRR as its routing stack, so the FRR configurations in this book carry over to SONiC's routing container almost unchanged. The MPLS **data plane**, however, depends on the switch hardware and its SAI implementation, and the containerised SONiC virtual switch has limited MPLS forwarding. For learning MPLS, plain FRR containers are the reliable choice; use SONiC when you need its management model or a specific hardware target.

The `frr.conf` files also work in any other environment: a GNS3 project, a set of VMs, or your own Docker tooling. Copy each router's file, start the daemons listed in `daemons`, and run the `exec` commands from the topology file by hand.

## Summary

- Labs run **FRR** containers under **containerlab**; the **Linux kernel** forwards labelled packets.
- Load **`mpls_router`** and **`mpls_iptunnel`** on the **host**; containers cannot load modules.
- Each container needs **`net.mpls.platform_labels`** set, and **`net.mpls.conf.<if>.input=1`** on each core interface. The topology files do this.
- The **core network** has pe1, pe2, p1, p2 and p3 running **IS-IS level 2**; shortest pe1 to pe2 path is along the top row.
- Customer sites **RED** and **BLUE** reuse the same addresses on purpose.
