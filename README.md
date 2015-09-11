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
