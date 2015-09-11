# Towards a Containerized Future
## Bryan Cantrill, CTO at Joyent

chroot (7th ed unix) -> bsd jails -> solaris zones
 -> security -> workload consolidation
OS virtualization requires OS modification (can't just everyone be on Solaris)

HW virtualization - OS level is hard, OS heavy, complex, HW virt became defacto standard, even in the 60s SW is heavier than HW
IBM1401 is a cockroach, virt is lie

Joyent OS level virt (container) since 2006
SmartOS (based on Solaris zones)
dotCloud: containers as PaaS foundations, docker

docker: appeals to developers
docker will do to apt what apt did to tar

deploying containers to VM is god is angry Wrong
pets vs cattle, docker hosts are pets even if your containers re cattle
next step: container native infrastructure

problems:
docker is linux centric
SmartOS solver lx-based zones, run linux binaries on SmartOS
Docker engine in Paas control plane - bad idea

the power of the interface
George Stephenson - railway gauge size, victorian hackernews
demo day, MP hit by train, funeral makes it viral

twistlock inventor, shipping container
standard interface is POWER

docker remote API, expressive, modern, robust,
triton: docker API -> SmartDataCenter, containers on metal

container landscape
it's the future, it's clear
upstack - fight, lots of choice, service discovery, composition, SDN
unlikely monopoly, all open source

not just compute, Joyent Manta storage service
break down many-to-one container-VM relationship
[VMWare Bonneville](https://blogs.vmware.com/cloudnative/introducing-project-bonneville/)

future:
on metal with multi-tenant security

# Introducing the Private Container Service
## Shannon Williams, Co-founder at Rancher Labs

organisation adoption
usually want some sort of container service
to allow more than just developer adoption

container service
* abstracting away:
    * computing resources (at different providers)
    * orchestration and SDN, discovery, LB, storage)
* allow integrating with ci/cd tools and config management

cloud66, AWS ECS, Rancher

# Distributed systems with (almost) no consensus
## Bryan Boreham, Weave

making a world a better place through paxos - no
'the server' is experiencing issues
anything with more than one container is distributed, to make it reliable
state has to be replicated
8 fallacies of dist systems by bill joy + 'hosts stay up forever'

what to do?

off the shelf: etcd, consul
gets complex, consensus algorithms: paxos, raft

cost of consensus:
* network roundtrips
* partitions (without majority) make you unavailable
> raftscope, weavescope

engineering: picking tradeoffs, weave relies on some other way

why weave cares:
service discovery via DNS
IP allocation

paxos, raft -> algorithms
how about data structure centric?

how important that all nodes are up-to-date?
serv discovery -> node crash, list of out-to-date
IP alloc -> most nodes don't care about all allocation

no consensus but eventual consitency

solution: CRDTs
* consistency without consesus
* updates can be merged in any order (merge: commutative,associative, idempotent)

CRDTs applied to DNS
each host only manipulates its own entries
delete with tombstone, (with timeout to prevent accumulating them)

CRDT for IP allocation
ring data structure, split by host


broadcast and gossip
* broadcast doesn't scale, n^2
* CRDT based systems often use gossip
* weave adapts gossip to work with narrow links between hosts

takeaways:
no consensus when you don't have to
CRDTs are great for eventual consistency
take great care when designing

Qs:
* IP alloc bootstrap, uses simple paxos, didn't need all features, simple paxos simpler than raft
* multicast is limited because of ops, eg. try that in AWS

# Kubernetes
## Mandy Waite, Google

how to make compute resources available to engineers?

dev view:
cell: cluster to run it in
binary: fat, statically linked, in container
args
resource requires
replicas

ops view:
bazel - google build system, open source
binary to cell storage
config to borg scheduler per cell, creates plan

prod vs nonprod priorities
nonprod - batch jobs, can get preempted

effeciency:
limits vs actual resource usage
dynamic reservation allows reusing requested but not reserved resources for lower priority tasks

cpu vs memory to be inline, don't waste free memory of CPU100% machine
matching machine shapes with workload shapes
workload shapes GB mem + core
computing tetris, bin packing

DC is a cell, machines are jsut resource boundaries

### Kubernetes

orchestrator for docker container, supports all major cloud providers, open source, go
manage apps not machines
kubernetes master is not HA replicated, but can run in GCE, hosted

Pod -> container or containers, or container + volumes
when it makes sense to run 2 container on the same host
pods have labels e.g version
and type: e.g service type
Replication controller makes sure expected state is maintained per label

Google Container Engine: hosted kubernetes, no need to worry about master HA

# LXD - The container lighter visor
## Stéphane Graber, Canonical

wrapper on top of LXC, simple REST API, command line tool
run full OS/system in container not app containers
their idea is to run CoreOs or your host with the docker engine in an LXD container
coming: safe live migration, lxd on snappy

# Docker network performance in the public cloud
## Arjan Schaaf, Luminis

* Kubernetes and CoreOS on Azure vs AWS
* qperf: short running test
* iperf3: longer running tests, parallel connections
* containers: arjanschaaf/centos-iperf3
* bandwith, latency - amazon is better

* options:
connect over the host interface
use sdn: weave, flannel, calico
before docker 1.7 - replace docker bridge or proxy in fron of docker deamon
1.7 - libnetwork

SDN functionality:
encryption & DNS (weave)
libnetwork and/or kubernetes support
Flannel & Weave - overlay
Calico - L2/L3

Flannel:
by CoreOS, easy setup, different backend, UDP, AWS VPC, VXLAN
weabe: DNS, proxy based, different backend: pcap, VXLAN
calico: vRouters connecte over BGP routes,
no overlay when running L2/L3 but IPIP tunnel on AWS (or plbic cloud)

bandwith: flannel VXLAN superfast, calico, weave
latency: weave is higher

native vs sdn: network bandwith + latency is not too bad
but CPU usage can get higher, e.g Flannel UDP but VXLAN still rocks
calico kind of in the middle
looking forward weave VXLAN
Calico upcoming nice kubernetes integration

Flannel VXLAN rocks

synthethic tests vs real test with applications
