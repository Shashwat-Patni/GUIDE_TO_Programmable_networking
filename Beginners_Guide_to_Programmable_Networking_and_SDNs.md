# Beginners Guide to Programmable Networking and SDNs

This is a self paced guide to cover the broad entry level topics in SDNs and Data plane
Programming. I recommend getting a good understanding of the terms involved and going deep as
and when required or whenever curiosity takes a hold of you. Most of the concepts do not
require a deep network related knowledge but a general starting point is to understand the
different terms of the OSI layer.

After learning these terms, a good point to start understanding what P4 is and what it enables
u to do is to go through the tutorials on the official repo. U will require a VM setup and a
basic understanding of simulations and emulations. Please note that this guide is meant to
provide u with links to official reading sources and a kickstarter to ur journey in the SDN
world.

**The path:**

1. Understanding networks
2. SDNs, control planes and data planes
3. The need for P4
4. P4 — its models, architectures and compilers
5. P4 tutorials for a hands on experience

**The main aim of all this is to understand and run the different P4 tutorials.** Sections 1 to 4
are there so u know what u are doing when u get to them, so don't get stuck reading — once the
terms make sense, move on to the tutorials and come back to the reading whenever u need it.

Most links below are short reads. Where something is long, it says so, and u only need it when
u go deep.

---

## 1. Understanding networks

Start with the OSI layers — almost every other term in this guide sits on one of them.

- [What Is The OSI Model?](https://www.cloudflare.com/learning/ddos/glossary/open-systems-interconnection-model-osi/)
  (Cloudflare) — the 7 layers, one by one, in plain words.
- [What is the network layer?](https://www.cloudflare.com/learning/network-layer/what-is-the-network-layer/)
  (Cloudflare) — layer 3, where IP and routing live.
- [What is a protocol?](https://www.cloudflare.com/learning/network-layer/what-is-a-protocol/)
  (Cloudflare) — short.
- [1.3 Architecture](https://book.systemsapproach.org/foundation/architecture.html) from
  *Computer Networks: A Systems Approach* — layering, encapsulation, the OSI model and how the
  Internet's own stack maps onto it.

| Term | What it is |
| --- | --- |
| Packet / frame | A unit of data on the wire. Ethernet carries frames; IP carries packets inside them |
| Header | The fields at the front of a packet that say where it is going and what it carries |
| MAC address | Hardware address, used inside one local network (layer 2) |
| IP address, subnet | Address used across networks (layer 3); a subnet is a block of addresses sharing a prefix |
| TCP / UDP | Transport protocols (layer 4): TCP is a reliable connection, UDP sends single messages |
| Port | A number identifying an application on a host — DNS uses 53, HTTPS uses 443 |
| Switch / router | A switch forwards frames inside a network; a router forwards packets between networks |
| Forwarding table | The lookup table a switch or router uses to decide where each packet goes |
| QoS | Treating some traffic better than other traffic: separate queues, priorities, rate limits |

Want to see these on real traffic? Install [Wireshark](https://www.wireshark.org/docs/wsug_html_chunked/)
and do the [Getting Started](https://www-net.cs.umass.edu/wireshark-labs/Wireshark_Intro_v9.0.pdf)
lab from Kurose & Ross.

---

## 2. SDNs, control planes and data planes

Every network device does two jobs: **deciding** where traffic should go, and **moving** each
packet there. SDN is the idea of pulling the deciding part out of the boxes and running it as
software.

- [What is the control plane?](https://www.cloudflare.com/learning/network-layer/what-is-the-control-plane/)
  (Cloudflare) — control plane vs data plane, with a good traffic-light analogy.
- [What is software-defined networking (SDN)?](https://www.cloudflare.com/learning/network-layer/what-is-sdn/)
  (Cloudflare) — what changes when the control plane is separated from the hardware.
- [1.3 SDN: A Definition](https://sdn.systemsapproach.org/intro.html#sdn-a-definition) from
  *Software-Defined Networks: A Systems Approach* — a precise definition, a few paragraphs.
- [Chapter 3, Basic Architecture](https://sdn.systemsapproach.org/arch.html) from the same book —
  how the pieces of an SDN stack fit together: switch, switch OS, network OS, applications.
- [ONOS](https://opennetworking.org/onos/) — what an open-source SDN controller (network
  operating system) looks like.

| Term | What it is |
| --- | --- |
| Data plane (forwarding plane) | The part of a switch that forwards every packet, at line rate |
| Control plane | The software that decides how packets should be forwarded and fills the data plane's tables |
| Line rate | The full speed of a port — the data plane has to keep up with every packet |
| SDN | Software-defined networking: the control plane runs as software, separate from the switches |
| Controller / Network OS | The central software that runs the control plane for many switches (e.g. ONOS) |
| Southbound API | How the controller talks down to the switches — e.g. OpenFlow, P4Runtime |
| Northbound API | How applications talk to the controller |
| OpenFlow | The first widely used southbound protocol: the controller adds and removes entries in a switch's tables |
| Bare-metal / white-box switch | A switch whose hardware and software come separately, so u can run ur own software on it |

---

## 3. The need for P4

OpenFlow lets a controller fill a switch's tables, but the switch still only understands the
headers its chip was built for. P4 goes one level lower: it lets u program **what the switch
does with packets** in the first place.

- [Clarifying the differences between P4 and OpenFlow](https://opennetworking.org/news-and-events/blog/clarifying-the-differences-between-p4-and-openflow/)
  (Open Networking Foundation blog, by Nick McKeown and Jennifer Rexford) — short, and the
  clearest explanation of why P4 exists.
- [4.1 Switch-Level Schematic and 4.2 Forwarding Pipeline](https://sdn.systemsapproach.org/switch.html)
  from *Software-Defined Networks: A Systems Approach* — what is inside a switch, and the
  parser → match-action stages → deparser pipeline that P4 programs.
- Bosshart et al., [P4: Programming Protocol-Independent Packet Processors](https://arxiv.org/abs/1312.1719)
  (ACM SIGCOMM CCR, 2014) — the original P4 paper, a few pages long.

| Term | What it is |
| --- | --- |
| P4 | A language for programming the data plane — what a switch parses, matches and does |
| Protocol-independent | The switch is not built for fixed headers; the P4 program defines which headers exist |
| Parser / deparser | Reads headers out of an incoming packet / writes them back into the outgoing one |
| Match-action table | "If these header fields match, do this action" — the basic building block of a P4 program |
| Action | What happens on a match: set a field, pick an output port, drop, and so on |
| Pipeline | The fixed sequence a packet goes through: parser, match-action stages, deparser |

---

## 4. P4 — its models, architectures and compilers

**None of the links in this section need to be read in full.** Skim them so u know what each
architecture, compiler and switch is, and come back to them when a tutorial (or later the
Tofino) needs it.

Three things to keep apart:

- **The language** (P4₁₆) — the same everywhere.
- **The architecture** — the shape of the pipeline a device offers (which blocks, in which
  order, and which built-in functions, called externs). Ur program is written *against* one,
  through an `#include` like `v1model.p4` or `tna.p4`.
- **The target** — the actual device that runs it, with its own compiler backend.

**Start here:** [4.3 Abstracting the Pipeline](https://sdn.systemsapproach.org/switch.html#abstracting-the-pipeline)
from *Software-Defined Networks: A Systems Approach* — explains PSA, v1model and TNA and why
there is more than one.

| Architecture | Made for | Defined by | Runs on |
| --- | --- | --- | --- |
| v1model | Learning and research; what the tutorials use | [`v1model.p4`](https://github.com/p4lang/p4c/blob/main/p4include/v1model.p4) in the p4c repo (no separate spec) | BMv2 `simple_switch` |
| PSA (Portable Switch Architecture) | A portable standard for switches | [PSA specification](https://p4lang.github.io/p4-spec/docs/PSA.html) | BMv2 `psa_switch` |
| PNA (Portable NIC Architecture) | Network cards (NICs) rather than switches | [PNA specification](https://p4.org/wp-content/uploads/sites/53/p4-spec/docs/PNA.html) | BMv2 `pna_nic`, DPDK |
| TNA (Tofino Native Architecture) | Intel's Tofino switch chip | [TNA public spec](https://github.com/barefootnetworks/Open-Tofino/blob/master/PUBLIC_Tofino-Native-Arch.pdf) (PDF) | Tofino |

**Compilers**

- [p4c](https://github.com/p4lang/p4c) — the reference P4 compiler. A shared front end checks
  and simplifies ur program; a [back end](https://github.com/p4lang/p4c/tree/main/backends) per
  target turns it into something that device can run. The repo has back ends for BMv2, DPDK,
  eBPF, uBPF, Linux tc and Tofino, plus `p4test` (front end only) and `graphs` (draws ur
  program's control flow).
- `bf-p4c` — the Tofino compiler that comes with Intel's switch software; see
  [open-p4studio](https://github.com/p4lang/open-p4studio), which also includes a Tofino
  simulation model u can run without the hardware.

**Software switch**

- [BMv2](https://github.com/p4lang/behavioral-model) — the reference software switch. Its
  [`simple_switch` docs](https://github.com/p4lang/behavioral-model/blob/main/docs/simple_switch.md)
  explain how the v1model pipeline actually behaves; the [targets](https://github.com/p4lang/behavioral-model/tree/main/targets)
  folder lists the other switch models (`simple_switch_grpc`, `psa_switch`, `pna_nic`, …).

**Talking to the switch at runtime**

- [Announcing P4Runtime](https://opennetworking.org/news-and-events/blog/announcing-p4runtime-a-contribution-by-the-p4-api-working-group/)
  (ONF blog) — short; what P4Runtime is and why it was needed.
- [P4Runtime specification](https://p4lang.github.io/p4runtime/spec/main/P4Runtime-Spec.html) —
  reference only, long.

**Optional:** Andy Fingerhut's [slides on P4 and the Portable NIC Architecture](https://research.cec.sc.edu/files/cyberinfra/files/P4-PNA-presentation-2024-aug.pdf)
compare v1model, PSA and PNA from the point of view of someone who works on them.

---

## 5. P4 tutorials for a hands on experience

This is the main part of the guide. The goal is to understand and run each of the tutorials
urself, not just read through them.

### Simulation vs emulation

- A **simulator** models a network in software and computes what would happen, e.g.
  [ns-3](https://www.nsnam.org/).
- An **emulator** runs real software (real hosts, real switch programs) on a virtual network,
  e.g. [Mininet](http://mininet.org/). The P4 tutorials use Mininet plus BMv2, so ur P4 code
  actually runs and forwards real packets inside ur laptop.
- [Introduction to Mininet](https://github.com/mininet/mininet/wiki/Introduction-to-Mininet) and
  the [Mininet Walkthrough](http://mininet.org/walkthrough/) explain how it works.

### VM setup

The [p4lang/tutorials](https://github.com/p4lang/tutorials) README has a pre-built VM with all
the P4 tools installed. U need VirtualBox and a 64-bit Intel/AMD machine or an Apple Silicon Mac.
The login is `p4` / `p4`. The README also has instructions for building ur own VM or installing
natively on Ubuntu 22.04 / 24.04.

If u are new to the Linux terminal, lectures 1 and 2 of
[The Missing Semester of Your CS Education](https://missing.csail.mit.edu/) (MIT) cover what u
need.

### The tutorials

The [official tutorials](https://github.com/p4lang/tutorials) are grouped into modules. Each
exercise has a README with the task, a `.p4` file with gaps to fill, and a `solution/` folder —
try it urself before opening the solution. Slides are in `P4_tutorial.pdf` in the repo.

1. **Introduction and language basics** — `basic` (IPv4 forwarding), `basic_tunnel`
2. **P4Runtime and the control plane** — `p4runtime`, `flowcache`
3. **Monitoring and debugging** — `ecn`, `mri`
4. **Advanced behaviour** — `source_routing`, `calc`, `load_balance`, `qos`, `multicast`
5. **Stateful packet processing** — `firewall`, `link_monitor`

`basic` is the one to start with. After that, go wherever ur curiosity takes u.

**More hands on:**

- [nsg-ethz/p4-learning](https://github.com/nsg-ethz/p4-learning) (ETH Zürich) — a large
  collection of exercises, examples and documentation.
- [P4 Learn](https://p4.org/learn/) — the P4 Language Consortium's own list of learning
  material.
- [p4-guide](https://github.com/jafingerhut/p4-guide) — a map of all the p4lang repositories,
  with a lot of good learning material.

**Keep open while coding:** the [P4₁₆ language specification](https://p4lang.github.io/p4-spec/docs/P4-16-working-spec.html) —
reference only, don't read it cover to cover.

---

## Books (optional)

For when u want the full picture of a topic rather than a short read.

| Book | Authors | Good for | Access |
| --- | --- | --- | --- |
| [Computer Networks: A Systems Approach](https://book.systemsapproach.org/) | Peterson & Davie | Networking fundamentals, from layers to QoS | Free online |
| [Computer Networking: A Top-Down Approach](https://gaia.cs.umass.edu/kurose_ross/index.php) | Kurose & Ross | The standard intro textbook; the site has lecture videos, slides and the Wireshark labs | Book is paid; site resources free |
| [Computer Networking: Principles, Protocols and Practice](https://www.computer-networking.info/) | Bonaventure | A second free intro textbook, with exercises | Free online |
| [Software-Defined Networks: A Systems Approach](https://sdn.systemsapproach.org/) | Peterson, Cascone, O'Connor, Vachuska & Davie | SDN, programmable switches, P4 and P4Runtime | Free online |
