![logo](logo.png)

# Towards a Containerized Future
## Bryan Cantrill, Joyent

### beginnings

* chroot (7th ed unix) -> bsd jails -> solaris zones
* beginning -> security -> workload consolidation


### hardware virtualization?

* OS virtualization is hard and not everyone is on Solaris
* HW virtualization - OS level is hard, OS heavy, complex,
* HW virt became defacto standard
* actually even in the 60s SW is heavier than HW

### containers strike back

Joyent OS level virt (container) since 2006
SmartOS (based on Solaris zones)
dotCloud: containers as PaaS foundations, docker

### docker: appeals to developers

### docker will do to apt what apt did to tar

### deploying containers to VM is 'god is angry' WRONG

* pets vs cattle
* docker hosts are pets even if your containers are cattle
* next step: container native infrastructure
* mentioned [VMWare Bonneville](https://blogs.vmware.com/cloudnative/introducing-project-bonneville/)

### current problems for Joyent

* docker is linux centric
* SmartOS solver lx-based zones, run linux binaries on SmartOS
* Docker engine in Paas control plane - bad idea

### aside: the power of the interface

* George Stephenson - railway gauge size, victorian hackernews
* demo day, MP hit by train, funeral makes it viral
* twistlock inventor, shipping container
* **standard interface is POWER**

### docker remote API

* expressive, modern, robust,
* triton: docker API -> SmartDataCenter, containers on metal

### container landscape
* it's the future, it's clear
* upstack - fight, lots of choice, service discovery, composition, SDN
* unlikely monopoly, all open source
* not just compute, Joyent Manta storage service

### break down many-to-one container-VM relationship
* future: on metal with multi-tenant security

# Introducing the Private Container Service
## Shannon Williams, Rancher Labs

### organisation adoption

* usually want some sort of container service
* to allow more than just developer adoption

### container service

* abstracting away:
    * computing resources (at different providers)
    * orchestration and SDN, discovery, LB, storage)
* allow integrating with ci/cd tools and config management

* cloud66, AWS ECS, Rancher

# Distributed systems with (almost) no consensus
## Bryan Boreham, Weave

> 'the server' is experiencing issues - nope

* anything with more than one container is distributed
* to make it reliable state has to be replicated somehow
* 8 fallacies of dist systems by bill joy + 'hosts stay up forever'

### what to do?

* off the shelf: etcd, consul
* gets complex, consensus algorithms: paxos, raft

### cost of consensus:
* network roundtrips
* partitions (without majority) make you unavailable

> mentioned: raftscope, weavescope

### engineering: picking tradeoffs

### why weave cares about consensus

* service discovery via DNS
* IP allocation
* paxos, raft -> algorithms
* how about data structure centric?

### how important that all nodes are up-to-date?

* serv discovery -> node crash, list of out-to-date
* IP alloc -> most nodes don't care about all allocation
* no consensus but eventual consistency is enough

### solution: CRDTs

* consistency without consensus
* updates can be merged in any order

### CRDTs applied to DNS
* each host only manipulates its own entries
* delete with tombstone, (with timeout to prevent accumulating them)

### CRDT for IP allocation
* ring data structure, split by host

### broadcast vs gossip

* broadcast doesn't scale, n^2
* CRDT based systems often use gossip
* weave adapts gossip to work with narrow links between hosts

### takeaways
no consensus when you don't have to
CRDTs are great for eventual consistency
take great care when designing

### Qs:
* IP alloc bootstrap, uses simple paxos, didn't need all features, simple paxos simpler than raft
* multicast is limited because of ops, eg. try that in AWS

# Kubernetes
## Mandy Waite, Google

### problem: how to make compute resources available to engineers?

### Borg - dev view
* cell: cluster to run it in
* binary: fat, statically linked, in container
* config/args
* resource limits
* no of replicas

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
Replication controller makes sure expected state is maintained per label - (X replicates of pods)

Google Container Engine: hosted kubernetes, no need to worry about master HA

# LXD - The container lighter visor
## Stéphane Graber, Canonical

* wrapper on top of LXC, simple REST API, command line tool
* run full OS/system in container not app containers
* their idea is to run CoreOs or your docker host in an LXD container
* coming: safe live migration, lxd on snappy

# Docker network performance in the public cloud
## Arjan Schaaf, Luminis

https://arjanschaaf.github.io/is-the-network-the-limit/

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

# Managing Kubernetes and OpenShift with ManageIQ
## Alissa Bonas, Red Hat

* containers on multiple hosts are the problem
* kubernetes + openshift + manageiq is RedHat's answer
* Openshift 3 - built on kubernetes
* manageiq collects and correlates information about nodes, pods
* allows things like verifying/browsing image sources, etc.
* future: package version analysis, etc

# Leveraging the DNS for fun and profit
## Miek Gieben, Improbable

### reactive infrastructure
* flexible, ops light, self-heal
* their stack: etcd, SkyDNS, ELK, Prometheus

### discovery
* SkyDNS: SRV records
* Prometheus: SRV record aware
* FQDN, resolv.conf search domains

### load balancing + health checks
* SkyDNS: round robin load balancing
* SkyDNS health signal: short ttl, dropped name

### registration
* newly started services have to register
* scheduler or docker host level could work
* but in-container can easily be health check based, no scheduler overhead, naturally with container lifecycle but requires multiprocess containers

### [dinit](https://github.com/miekg/dinit)
* super light init system
* also solves zombie-reaping problem

### runtime config
* flagz - dynamic flags from etcd

# Lessons learnts from running potentially malicious code
## Ben Hall, Ocelot Uproar

* potential security issues
* when running untrusted code in containers (see [scrapbook](http://www.joinscrapbook.com/))

### CPU
* `--ulimit nproc` to prevent fork bombs

### disk
* watch out for ways to fill your disks: logging, fallocate, truncate, dd, also etc/hosts mounted from host!
* set quotas for container directories

### network
* `-icc=false` no inter container communications
* never allow `--net=host` - e.g. shutdown shuts down host!
* restricting bandwidth is tricky
* users running ngrok, tor, torrent, ddos attacks

### troubleshooting/monitoring
* docker diff, bash_history, sysdig
* coming: the warden from Ocelot Uproar

# Docker Content trust
## Diogo Mónica, Docker

### The Update Framework (TUF)
* security framework
* protects against
    * replay attacks - serving stale content (everything has an expiration)
    * protects against key compromise (online vs offline key)
    * protect against mix and match attacks (signed collections vs. signed objects)

### notary implements TUF
* not docker specific
* separate notary-signer server holding timestamp keys
* notary server serving content only has public data
* only publisher has the rest of the keys
* transparent key-rotation

### docker content trust: built on notary
* uses pull-by-digest (registry v2 is content addressable)
* manifest -> hash chain, then pull by digest the rest of the layers
* enable it with `DOCKER_CONTENT_TRUST=1`
* or pull `--disable-content-trust=false`
* default docker cli was kept simple
* notary cli allows key rotation, etc.

### future: on by default

# Container visibility
## Loris Degioanni, Sysdig

* containers are great but inspecting them is not easy
* options: command line, cadvisor, stats API, sysdig

### command line
* resource usage: ps/top/htop
* top supports CGROUPS field
* network: iftop/tcpdump/tshark
* disk: iotop/lsof
* docker ps, docker top (supports ps options)

### cadvisor
easy to install
nice integration with heapster (kubernetes)
not too many metrics

### docker stats
* cloudwatt/docker-collect-plugin
* bit richer than cadvisor but not much more

### sysdig
* kernel module has to be installed
* metrics can be saved to a trace file
* very much tcpdump like
* scriptable
* curses interface like htop
* amazing, you gotta see a demo

# Containers at Hyperspeed
## Protocol Labs, Juan Batiz-Benet

### centralized infrastructure is less than ideal
* companies rely on docker hub, github
* DDoS
* wasting bandwidth

### the web
* things doesn't work when offline
* wasting bandwidth

### IPFS - making the web p2p
* can we take some ideas from git and bittorrent?
* merkledag or hash-chain - with content addressed data
* honeybadger don't care
* [blog post here](https://ipfs.io/ipfs/QmNhFJjGcMPqpuYfxL62VVB9528NXqDNMFXiqN5bgFYiZ1/its-time-for-the-permanent-web.html)

### containers+IPFS = starship
* container images can get big (unless you use alpine :))
* starship: IPFS for distributing container images
