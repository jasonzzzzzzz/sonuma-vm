## Overview
Sonuma-vm is an all-software, proof-of-concept implementation of Scale-Out NUMA, a high-performance rack-scale system for in-memory data processing. Scale-Out NUMA layers a one-sided (RDMA-like) protocol for explicit access to remote memory (with copy semantics) in lieu of a cache-coherence protocol on top of the NUMA's cache-block request/reply transpors. A specialized on-chip hardware block called remote memory controller (RMC) implementes the basic one-sided primitives (e.g., remote read/write).

Sonuma-vm is envisioned to be a software development platform that is capable of running Scale-Out NUMA applications at native machine speed and that can approximate the remote access latency of an actual hardware RMC. Sonuma-vm emulates a non-coherent NUMA machine with explicit, one-sided access to remote memory (i.e., Scale-Out NUMA) by combining a fully-coherent NUMA machine (ccNUMA) and a virtual machine monitor (hypervisor). Sonuma-vm emulates a multi-node Scale-Out NUMA system using a group of virtual machines (VM) mapped (VCPU, page frames) to distinct NUMA domains of a giant ccNUMA machine. Sonuma-vm leverages the Xen hypervisor to enable remote access across different NUMA domains. The RMC is implemented in software and runs on a dedicated VCPU within each VM. Sonuma-vm can be used with hardware virtualization enabled (CPU and IO) to minimize the emulation overheads. The platform comes with a library exposing the Scale-Out NUMA API from the ASPLOS'14 paper. 

## Requirements

- Intel or AMD ccNUMA server
- Intel x520 (SRIOV)
- Xen hypervisor

## Setup instructions:
```
export LIBSONUMA_PATH="path to libsonuma"<br/>
make (output in ./bin)<br/> 
```

<br />
#server 0<br />
insmod ./rmc.ko mynid=0<br />
#server 1<br />
insmod ./rmc.ko mynid=1<br />
<br />

#server 0<br />
taskset -c 1 ./rmcd 2 0 &<br />
#server 1<br />
taskset -c 1 ./rmcd 2 1 &<br />
<br />

## Micro-benchmarks: read/write from remote memory using sync/async operations:
#server 1 (write values to the context)<br />
taskset -c 0 ./bench_server<br />
#server 0 (read values from the memory of server 1<br />
taskset -c 0 ./bench_sync 1 r<br />
#server 0 (zero out the memory of server 1, read again to check)<br />
taskset -c 0 ./bench_sync 1 w<br />
#sever 0 (read from remote memory asynchronously)<br />
taskset -c 0 ./bench_async 1 r<br />
#sever 0 (write remote memory asynchronously)<br />
taskset -c 0 ./bench_async 1 w<br />
<br />

### sonuma-vm limitations:
This platform trades flexibility for performance. In particular, a Scale-Out NUMA application is bound to a single global address space, which is sufficient for the majority of use cases. Also, the size
of the context is fixed by the RMC daemon and the kernel driver. One could try to increase the context size (these are just constants), but it is not guaranteed that the Xen hypervisor will grant access
to all the pages.