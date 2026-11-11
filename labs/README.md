# Labs for the Concise Book on MPLS

Each folder is a self-contained [containerlab](https://containerlab.dev) lab built from
FRRouting containers (`quay.io/frrouting/frr:10.7.0`) and small Alpine hosts.

| Folder | Chapter | What it builds |
|---|---|---|
| `lab1-static-lsp` | 2 | Three routers and two hosts; one LSP built by hand with static labels |
| `lab2-ldp` | 4 | Five-router core: IS-IS plus LDP |
| `lab3-bgp-free-core` | 5 | Lab 2 plus iBGP between the PEs and two customer sites |
| `lab4-l3vpn` | 6 | Lab 2 plus VRFs RED and BLUE with overlapping addresses |
| `lab5-sr-mpls` | 9, 10 | Lab 4 with SR-MPLS (IS-IS) instead of LDP, plus TI-LFA |
| `lab6-sr-te` | 11 | Lab 5 core with an SR-TE policy on pe1 |
| `lab7-srv6` | 12 | IPv6-only core with SRv6 (uSID) carrying an IPv4 L3VPN |

## Requirements

* A Linux host (a VM, a server, or WSL2) with Docker and containerlab.
* The kernel's MPLS modules, loaded on the **host** before deploying:

```bash
sudo modprobe mpls_router mpls_iptunnel
ls /proc/sys/net/mpls          # must exist, or MPLS labs will not deploy
```

* For `lab7-srv6`, a kernel of 6.1 or newer (`uname -r`).

## Running a lab

```bash
cd lab2-ldp
sudo containerlab deploy -t lab2-ldp.clab.yml
docker exec -it clab-lab2-ldp-pe1 vtysh
sudo containerlab destroy -t lab2-ldp.clab.yml --cleanup
```

## Using the files elsewhere (DockerLab, GNS3, VMs)

* `configs/<node>/frr.conf` is a complete FRR configuration for that router.
* `configs/daemons` lists which FRR daemons to start.
* The `exec:` lines in each `.clab.yml` are the Linux commands each node needs after its
  interfaces exist (enable MPLS input per interface, create VRFs). The `sysctls:` block lists
  the kernel settings every node needs. Run the same commands in any other environment.
