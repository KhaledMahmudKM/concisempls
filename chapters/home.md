# Concise Book on MPLS

A concise, diagram-rich guide to label switching, from LDP to Segment Routing.

<img src="cover.svg" alt="Cover Image" width="100%" >

Almost every service provider, and a good share of large enterprises and cloud backbones, moves traffic with **MPLS**. It turns a routed IP network into a set of tunnels, and those tunnels carry VPNs for thousands of customers, steer traffic away from congested links, and repair failures faster than routing protocols can react.

This book explains how it works and why it is built the way it is. It assumes you know IP routing well: addressing, longest-prefix match, an IGP such as OSPF or IS-IS, and the basics of BGP. It assumes no prior MPLS knowledge.

Every core idea comes with a lab you can run on a laptop. The labs use **FRRouting** (FRR) containers in **containerlab**, so the routers you configure are real routing software running on the real Linux MPLS data plane.

---

## How this book is organized

**Part I, Foundations.** Why MPLS exists, what a label is and what routers do with it, and how to build the lab.

**Part II, Label Distribution.** How routers agree on labels with LDP, and how that lets the core of the network forget about BGP altogether.

**Part III, MPLS VPNs.** BGP/MPLS Layer 3 VPNs in depth, the design patterns operators use with them, and Layer 2 VPNs: pseudowires, VPLS and EVPN.

**Part IV, Segment Routing and Traffic Engineering.** SR-MPLS, which replaces LDP with a few extensions to the IGP; fast reroute with TI-LFA; traffic engineering with RSVP-TE and SR policies; and SRv6, which carries the same ideas over plain IPv6.

The **Appendix** collects the commands used in the labs, and the **Glossary** defines every term.

---

## How to read it

Read Parts I and II in order; everything later builds on them. After that, Part III and Part IV are largely independent.

Each chapter ends with a short summary. Lab sections are marked **Hands on**. A few topics, such as RSVP-TE and VPLS, have no working data plane in Linux, so they are explained with diagrams and marked **No lab**; the nearest lab you can run is suggested instead.

The lab files are in the [`labs/` folder](https://github.com/khaledmahmudkm/concisempls/tree/main/labs) of this book's repository.
