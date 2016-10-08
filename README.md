# Container Camp London 2015

**Update**: [videos are up here](https://www.youtube.com/playlist?list=PLcHZXHMeDzxUrNpD2Tms-zrZn9etw6JcQ).

I was at the amazing Container Camp (thanks to [YLD!](https://www.yld.io/)) and had a lot of fun. My raw notes are [on github](https://github.com/csabapalfi/container-camp-ldn-2015). See short talk summaries below.

* [Bryan Cantrill, Joyent - keynote ★](#bryanc)
* [Shannon Williams, Rancher](#shannon)
* [Bryan Boreham, Weave - CRDTs ★](#bryanb)
* [Mandy Waite, Google - Kubernetes](#mandy)
* [Stéphane Graber - LXD](#stephane)
* [Arjan Schaaf - Networking Performance](#arjan)
* [Alissa Bonas - OpenShift, ManageIQ](#alissa)
* [Miek Gieben - SkyDNS, dinit ★](#miek)
* [Ben Hall - Container Security ★](#ben)
* [Diogo Mónica - Docker Content Trust ★](#diogo)
* [Loris Degioanni - Sysdig ★](#loris)
* [Juan Batiz-Benet - IPFS, starship ★](#juan)

## <a id="bryanc"></a> Bryan Cantrill, Joyent - keynote ★

Bryan started the conference with an energetic keynote. Took us from the beginning of chroots to Solaris zones. Then he explained how hardware virtualization still became the de-facto standard and of course we ended up at docker.

Funny how we're still replaying history when running containers on VMs though. The future is **containers on bare metal** (container-native infrastructure) with multi-tenant security solved somehow.

## <a id="shannon"></a> Shannon Williams, Rancher

Shannon talked about **organization adoption** of docker. Most of them just want a container service abstracting away computing resources and orchestration.

## <a id="bryanb"></a> Bryan Boreham, Weave - CRDTs ★

**Achieving consensus is costly** in distributed systems (network roundtrips, availability). Weave needs to deal with distributed state to handle **IP allocation and service discovery with DNS**. Turns out consensus (and using etcd, consul with raft) is not necessary and **eventual consistency** is enough for their case.

They use **[CRDTs](https://en.wikipedia.org/wiki/Conflict-free_replicated_data_type)**. Just choose your data-structure well and make sure merging in updates can happen in any order.

Great talk highlighting how engineering is all about choosing the right trade-offs.

## <a id="mandy"></a> Mandy Waite, Google - Kubernetes

How to make compute resources available to engineers? **[Kubernetes](http://kubernetes.io/)** is Google's answer. It reached 1.0 in July and let's you manage apps not machines.

Kubernetes scheduling and controller components are **not highly-available yet** (in progress). For now you can trust Google Container Engine to run them for you.

## <a id="stephane"></a> Stéphane Graber, LXD

[LXD](http://www.ubuntu.com/cloud/tools/lxd) is a wrapper on top of LXC, simple REST API, command line tool. It's aimed at running full OS/system in container not app containers. Their idea is to run CoreOs or your docker host in an LXD container.

## <a id="arjan"></a> Arjan Schaaf - Networking Performance

Arjan Schaaf from Luminis was measuring **networking performance** of a Kubernetes and CoreOS setup. He compared various Azure and AWS instance types and **Weave, Calico and Flannel** SDNs.

**qperf** is great for quick tests. **[iperf3](https://github.com/esnet/iperf) is better for long running tests with parallel connections**. Bandwidth, latency and CPU was measured and **Flannel with VXLAN** was the winner. Weave is also working on VXLAN backend which sounds promising.

Arjan advises against **relying only on synthetic tests**. You  should really test using your application, too.

## <a id="alissa"></a>Alissa Bonas, Red Hat - OpenShift

**[OpenShift 3](https://www.openshift.com/)** is built on Kubernetes but also adds some higher level abstractions.

**ManageIQ** collects and correlates information about nodes, pods and the hosts running them. It also allows things like monitoring and security auditing.

## <a id="miek"></a> Miek Gieben - SkyDNS, dinit ★

Miek showed how Improbable builds it's reactive - **flexible, ops light and self-healing** - infrastructure. Their stack includes **etcd, SkyDNS, ELK, Prometheus**.

[SkyDNS](https://github.com/skynetservices/skydns) can solve discovery, simple load balancing and basic health signals. It also integrates well with Prometheus.

[dinit](https://github.com/miekg/dinit) is a super-light init system. Solves the zombie-reaping problem. It can also be used to enable in-container registration.

## <a id="ben"></a> Ben Hall - Container Security ★

Ben told us about lessons learnt while building [scrapbook](http://www.joinscrapbook.com/). It's training environments allow running arbitrary code by learners in containers.

Few interesting lessons included `--ulimit nproc` to disarm fork bombs. I also didn't know that `--net=host` allows **shutting down the docker host from a container**. It's worth watching out for ways how **a container can fill your disks**: logging, fallocate, truncate, dd. Also etc/hosts mounted from host and can be filled with garbage. **Network bandwith** -in and out- is also worth keeping an eye on.

Tools to help recognizing malicious activity include docker diff, bash_history or sysdig. Also **The Warden** is an upcoming tool from Ocelot Uproar.

## <a id="diogo"></a> Diogo Mónica - Docker Content Trust ★

Diogo is the security lead at docker and talked about new docker 1.8 feature. It's called content trust and allows **verifying** that an image is **up-to-date** and from the **right publisher**.

He started by describing **[The Update Framework (TUF)](http://theupdateframework.com/)** for secure software/content updates. It protects against:

* replay attacks (serving stale version) - by requiring expiration
* key compromise - by having a separate, offline root key
* mix and match attacks - by using signed collections
* and more...

**[Notary](https://github.com/docker/notary)** is an opinionated implementation of TUF by Docker Inc.

**[Docker Content Trust](https://docs.docker.com/security/trust/content_trust/)** is built on top of notary. You can give it a go by setting `DOCKER_CONTENT_TRUST=1` or specifying `--disable-content-trust=false`. In the future this will be on by default.

## <a id="loris"></a> Loris Degioanni - Sysdig ★

**Inspecting containers** resource usage, network and disk stats is not easy. Resources can be monitored with standard command line tools but sometimes difficult to get the right bits of information. **cAdvisor** is easy to install but doesn't have too many metrics. The **docker stats API** is a bit richer but sometimes even that is not enough.

**[Sysdig](http://www.sysdig.org/)** requires a **kernel module** but it's super powerful. Metrics can be saved to a **trace file** and processed later (just like tcpdump). It also has a nice **htop-like interface**. You have to see a demo.

## <a id="juan"></a>Juan Batiz-Benet - IPFS, starship ★

Juan from [Protocol Labs](http://ipn.io/) talked about how centralized infrastructure is less than ideal. Companies **rely on docker hub, github** and we all saw what a **DDoS** attack can do. We're also **wasting bandwidth** by downloading the same content from these central sources.

**[IPFS](https://ipfs.io/)** is about switching the web to a peer-to-peer protocol. It borrows some ideas from git and bittorrent and removes the need for a central server altogether. See their [blogpost here](https://ipfs.io/ipfs/QmNhFJjGcMPqpuYfxL62VVB9528NXqDNMFXiqN5bgFYiZ1/its-time-for-the-permanent-web.html).

**starship** is a solution to make pulling and pushing images faster and more reliable by using IPFS.
