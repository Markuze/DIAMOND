# Problem statement

Draft for §2, Background and motivation. Conventions are in
[outline.md](outline.md). Paper prose comes first; author notes follow.

## 2.1 Services that move bytes they do not read

A large class of network services sits between a producer and a consumer of
data and moves that data unchanged. An L7 load balancer reads a request
header, chooses a backend and relays the body. A caching proxy or CDN node
stores a response and serves it again. A pub/sub broker or media relay
delivers one message to many subscribers. A key-value cache stores a value and
returns it. A storage gateway, such as an NFS or object front end for a
distributed store, turns one protocol's payload into another's.

We call these *pass-through services*. They read a small part of each message,
typically tens to hundreds of bytes of header, to decide what to do. The
payload is opaque to them: they forward it, hold it, or both.

Such services carry much of the traffic in large deployments. At Facebook,
CDN caches serve 70% of web requests, and one cache server can replace tens
of database servers [cachelib]. Twitter runs hundreds of in-memory cache
clusters [twitter20], and Google's CliqueMap cache holds over a petabyte of
DRAM and serves about 150 million queries per second [cliquemap].
Microservices spend as little as 18% of their CPU cycles on core application
logic [accelerometer], and the service-mesh sidecars that proxy their
traffic add up to 269% to request latency and up to 163% more virtual CPUs
[meshinsight].

TODO(measure): the Stage 0 numbers beside these
(`../critisizm/bar_raizer_2026-10-03.md`, R9).

The mechanism this paper proposes applies where three conditions hold: the
length of a payload is known before it arrives, so that it can be taken by
reference as one unit; the application does not read it; and it reaches the
kernel in the clear, directly or after kernel TLS (§4). HTTP/1.1 bodies with
a content length, RESP and memcached values, log segments and the chunk
values of an inference cache meet all three. Chunked encoding and
multiplexed protocols such as HTTP/2 meet them at frame granularity.
TODO(decide): whether HTTP/2 is claimed.

Today these services are written against sockets, and sockets require the
payload to pass through the application. That requirement costs CPU and it
costs memory.

**A current instance: KV-cache stores for LLM inference.** Inference engines
keep the attention key and value tensors computed for a prompt so that later
requests sharing the prefix skip recomputation. This "KV cache" is large: for
models with 128K-token contexts it "can demand hundreds of gigabytes"
[directkv]. How large depends on the architecture: a Llama 3.1 8B cache is
128 KiB per token, DeepSeek-V3's latent attention needs about 70 KB, and
DeepSeek-V4.1-Flash reports about 890 bytes [dsv3hw, dsv41], so a store's
values range from tens of megabytes per 256-token chunk to a few hundred
kilobytes. Serving systems therefore move it off the GPU and across the
network. CacheGen compresses and streams KV caches between machines
[cachegen]; Mooncake builds a disaggregated serving architecture around a
distributed KV-cache store [mooncake]; LMCache stores KV caches in a hierarchy
that includes remote stores such as Redis [lmcache]. At the frontier,
DeepSeek's serving stack persists compressed KV to SSD with a guaranteed
lifetime of at least 72 hours for prefix reuse and keeps a cluster-wide pool
of host memory for the rest; its authors call KV cache management "a
foundational capability for model deployment" and note that "I/O and
interconnect bandwidth also limit cache migration and loading" [dsv41]. In
one deployment report the stored chunks are 500 KB to 40 MB each, a single 10,000-token request
produces 1.2 GB of them, and the store must deliver at least 2 GB/s for
fetching to beat recomputation [rediskv]. The store is a pass-through service
in its purest form: values are opaque tensors, written once, never modified,
and read by many inference nodes that share a prefix.

## 2.2 Cost one: CPU

Each payload byte is copied from the kernel into the application on receive
and back into the kernel on send. The copies do no useful work for a
pass-through service, and at current link speeds they dominate. A measurement
study of the Linux stack at 100 Gbps finds that "a single core is no longer
able to process packets at line rate, with data copy from kernel to
application buffers at the receiver becoming the core performance bottleneck"
[cai21]. Fleet-wide profiling attributes 22–27% of all cycles to a "datacenter
tax" that includes data movement [kanev15]. Two recent systems accept
copying as a first-class operating-system problem: zIO removes copies that
unmodified applications never needed [zio], and Copier turns copying into an
asynchronous kernel service [copier]. Each reports up to 1.8× application
throughput on Linux. A concurrent study of production proxies attributes up
to 55.6% of nginx and HAProxy cycles to copies across the kernel boundary,
and finds the receive-side copy the costlier one, because it walks scattered
packet fragments and misses the TLB ten times as often as the transmit side
[libra].

The cost grows with each pass over the payload and with each receiver. An
encrypted HTTP/2 load balancer handles each DATA frame up to five times: three
copies, a decryption and an encryption. Passing DATA frames through as buffer
lists, with TLS offloaded to the NIC, doubles its throughput for 10 KB and
100 KB responses [zcl7lb]. A userspace pub/sub broker makes one system call
and one pass through the network stack per subscriber; moving its fan-out
into the kernel triples throughput at 128 subscribers [bpfbroker]. The copy
is not always the largest cost. In the 100 Gbps study, copying dominates for
messages of 16 KB and more, while at 4 KB protocol processing and scheduling
cost more [cai21].

TODO(measure): share of CPU spent copying in a stock L7 proxy and a stock
key-value cache on our testbed, for payloads from 1 KB to 1 MB.

## 2.3 Cost two: memory

The same bytes are also stored several times. While a message is in flight
through a pass-through service it occupies the kernel receive queue, the
application's buffer and the kernel send queue. Table 1 lists the defaults
that size these buffers.

Table 1: buffer defaults per connection.

| Layer | Setting | Default |
| --- | --- | --- |
| Linux TCP receive buffer | `tcp_rmem` | 131072 bytes initially; autotuned up to between 131072 bytes and 32 MB |
| Linux TCP send buffer | `tcp_wmem` | 16 KB initially; autotuned up to between 64 KB and 4 MB |
| nginx proxy | `proxy_buffer_size`, `proxy_buffers` | one 4–8 KB buffer plus eight 4–8 KB buffers |
| HAProxy | `tune.bufsize` | 16384 bytes per buffer |
| Envoy | `per_connection_buffer_limit_bytes` | 1 MiB soft limit, for read and for write buffers |

Holding data multiplies the cost again. A cache keeps one copy of an object,
and every connection that is currently receiving the object keeps another in
its send queue. With N concurrent receivers of an object of size S, the cache
holds S bytes and the send queues hold up to N × min(S, send buffer) more.
Redis documents that these client output buffers can lead to data eviction
or out-of-memory errors, and that ordinary clients have no output-buffer
limit by default [redisclients]. Duplication in flight can therefore cost
cached data, not only memory.
A file served with `sendfile` avoids this, because the send queues refer to
page-cache pages. Data that arrived from the network has no such path: it must
first be copied into a file.

Prior work on copy elimination reports CPU and throughput. None of it reports
memory against the number of readers of a payload or against its reuse.

TODO(measure): resident and kernel socket memory per proxied connection at
10k–1M connections; memory against N for a hot object served from an in-memory
cache.

## 2.4 Why existing mechanisms fall short

**Zero-copy receive into application memory** removes the receive copy by
placing payload where the application can address it: `io_uring` zero-copy
receive [zcrx], device-memory TCP [devmem], page-remapping receive,
netmap-based stacks [paste], and designs that add I/O-only memory to the
kernel [maio]. All of them inherit one constraint. A page that is mapped into
an application must hold only that application's data and no kernel headers.
The destination of a packet must therefore be known before the packet is
stored, which requires header/data split, flow steering to dedicated queues,
driver changes, or changes to the kernel's memory manager.

For `io_uring` zero-copy receive, the one of these in mainline Linux, the
constraint limits a pass-through service in three ways. It registers only on
drivers that split headers and fill a dedicated queue from the registered
memory: in mainline today Broadcom's bnxt, Google's gve, NVIDIA's mlx5 and
Meta's fbnic, and not AWS's ENA, Azure's netvsc and MANA, virtio-net or veth
[zcrxdrivers]. On a capable NIC, payload that misses the steered queue is
copied into the pool instead [zcrxcopy], and loopback and veth, which carry
the traffic of sidecars and other same-host proxies, have no queue to
register at all. Each application needs its own queue and steering rules,
so the queues of a NIC bound how many applications can use it. These two
limits come from placing payload in application memory, and more software
would not remove them. The third is softer: the registered memory is also
the receive pool, so memory for data an application keeps is pinned up
front for the peak instead of charged as it is kept.

**In-kernel transfer** (`splice`, sockmap, socket splicing) moves payload from
one socket to another without the application. It transfers once. There is
nothing left to hold, to send again, or to send to a second socket.

**The page cache with `sendfile`** does hold data once and send it many times.
It needs a copy to bring network data in, and it represents each object as a
file.

**Copy optimization** [zio, copier] makes copies rarer, lazier or
asynchronous. The bytes still end up in application buffers, and the memory
picture of §2.3 does not change.

Table 2 in `novelty.md` (the property matrix) summarizes these.

## 2.5 Requirements

A mechanism for pass-through services should meet all of the following.

- **R1. No payload copy** on receive or on send.
- **R2. One physical copy**, however many connections are using the payload
  and however long it is held.
- **R3. Lifetime independent of connections.** Payload can outlive the socket
  it arrived on and be sent on sockets that did not exist when it arrived.
- **R4. Control stays in the application.** The application decides what
  happens to each payload and can read any byte range it asks for.
- **R5. Stock drivers, any NIC, no per-application hardware setup.** No
  hardware features, no driver changes, no queue or steering rule per
  application.
- **R6. Bounded, accounted memory.** Held payload is charged to its holder,
  and the memory pinned per payload byte is bounded.

No existing mechanism meets R1–R5 together (§8). R6 is the requirement that
holding network buffers creates, and §4 addresses it.

---

## Author notes

### Evidence still to collect

| Claim in this section | What backs it |
| --- | --- |
| Copies dominate CPU for pass-through services | TODO(measure): our own profile; [cai21], [libra], [zio], [copier], [zcl7lb] and [bpfbroker] as external support; [cai21] also bounds it to messages of 16 KB and more |
| In-flight bytes are stored three times | TODO(measure): per-connection memory; Table 1 is configuration, not measurement |
| Send queues duplicate a hot object N times | TODO(measure): memory against N |
| Zero-copy receive is unavailable, or forces a copy, for pass-through services | TODO(verify): a driver census from the kernel tree at the submission version (drivers with the queue-management API, a memory provider and header split); TODO(measure): the portability table's availability columns, with loopback and veth rows (bar-raiser R5). The holding and density limits follow from the documented interface [zcrx]; `outline.md` §7 Q2 measures them on a header-split NIC (required, the owner, 2026-10-06) |
| Prior copy-elimination work does not report memory against readers or reuse | TODO(verify) against the full text of [zio] and [copier]; Libra's appendix discusses socket-memory accounting and a per-connection cap, which is why the claim is narrowed to readers and reuse |

### Facts checked, with sources

- [cai21] Cai et al., "Understanding Host Network Stack Overheads", SIGCOMM
  '21. Quote is from the abstract.
  https://www.cs.cornell.edu/~ragarwal/pubs/network-stack.pdf
- [kanev15] Kanev et al., "Profiling a Warehouse-Scale Computer", ISCA '15.
  22–27% tax figure; the share of data movement alone is TODO(verify).
  https://research.google.com/pubs/archive/44271.pdf
- `tcp_rmem`, `tcp_wmem`: https://docs.kernel.org/networking/ip-sysctl.html
- nginx: https://nginx.org/en/docs/http/ngx_http_proxy_module.html
- HAProxy `tune.bufsize` default 16384. How many buffers a stream holds is
  TODO(verify). https://docs.haproxy.org/
- Envoy: https://www.envoyproxy.io/docs/envoy/latest/api-v3/config/listener/v3/listener.proto
- [zio] and [copier]: 1.8× figures from their abstracts.
- [libra] Zhou, Lin, Zhang, Zhao, arXiv 2604.27686 (2026-04-30), unrefereed.
  The 55.6% figure is Figure 1a; the TLB ratio is §2.1 and Figure 1b.
  Verified against the preprint text (`../refs/libra-arxiv2604.27686v1.pdf`).
- [dsv41] DeepSeek-AI, "DeepSeek-V4.1-Flash: Pushing the Limits of KV Cache
  Compression", arXiv 2609.19969v1 (2026-09-17), unrefereed; local copy
  `../refs/dsv41-arxiv2609.19969v1.pdf`. Verified against the text: 890 bytes per
  token in HBM and 1/4 of V4-Flash (abstract, §6); persistent footprint 1/8
  (abstract, §3.2.1); SSD cache with a guaranteed lifetime of at least 72
  hours, LRU, and the host-DRAM pool at 10% per machine with minute TTLs
  (§3.2.1); the quotes on KV cache management and bandwidth (§1); 437-fold
  reduction since V1 (Figure 1(b) caption).
- [redisclients] Redis client handling documentation, output-buffer limits
  and client eviction. Verified in `../critisizm/bar_raizer_2026-10-03.md`
  (R2 item 6) against https://redis.io/docs/latest/develop/reference/clients/
  ; re-check before LaTeX.
- [zcrx] Linux kernel documentation, "io_uring zero copy Rx"
  (https://docs.kernel.org/networking/iou-zcrx.html): header/data split, flow
  steering and RSS are required; buffers return to the pool through a refill
  ring. Requirements verified earlier (`../RELATED_WORK.md` §A).
- [zcrxdrivers] Driver census of the `torvalds/linux` master tree,
  2026-10-06, by grepping each driver for `queue_mgmt_ops` and the TCP
  data-split ring flag (`../critisizm/fable_intro_review_2026-10-06.md` §2):
  bnxt, gve, mlx5, fbnic and netdevsim have the queue API; ice, idpf, i40e,
  ixgbe, e1000, nfp, virtio_net, ENA, netvsc, mana and veth do not. It
  confirms the earlier web answer (bnxt, gve, mlx5 from 6.17) and adds
  fbnic. gve is Google Cloud's virtual NIC. TODO(verify) against a
  checked-out tree at the submission kernel.
- [zcrxcopy] "io_uring/zcrx: add CQE based notifications", Léger and
  Begunkov, v2, 2026-05-18: notifications and counters for the copy fallback
  taken when traffic arrives on a non-zcrx queue. TODO(verify) the fallback
  in the mainline source.
- [cai21], continued: copying dominates RPC processing from 16 KB messages;
  at 4 KB, TCP/IP processing and scheduling dominate (§3.7). Verified against
  the paper (`../refs/hoststack-sigcomm21.pdf`).
- [cachelib] Berg et al., "The CacheLib Caching Engine: Design and
  Experiences at Scale", OSDI '20. CDN caches serve 70% of web requests; one
  cache server replaces tens of database servers (§1).
  `../refs/cachelib-osdi20.pdf`, light review.
- [twitter20] Yang, Yue, Rashmi, "A large scale analysis of hundreds of
  in-memory cache clusters at Twitter", OSDI '20; 153 clusters analysed.
  `../refs/twittercache-osdi20.pdf`.
- [cliquemap] Singhvi et al., "CliqueMap: Productionizing an RMA-Based
  Distributed Caching System", SIGCOMM '21. Over 1 PB of DRAM, about 150M
  queries/s (§1). `../refs/cliquemap-sigcomm21.pdf`.
- [accelerometer] Sriraman et al., "Accelerometer: Understanding
  Acceleration Opportunities for Data Center Overheads at Hyperscale",
  ASPLOS '20. The 18% figure is from the abstract, verified in
  `../critisizm/bar_raizer_2026-10-03.md` (R9); not in `../refs/`.
- [meshinsight] Zhu et al., "Dissecting Overheads of Service Mesh Sidecars",
  SoCC '23. Abstract. `../refs/sidecars-socc23.pdf`.
- [zcl7lb] Podleś, Tafur Monroy, Vegas Olmos, Pismenny, "Towards encrypted
  zero-copy L7 load balancing", ANRW '26. Five passes per DATA frame (§1,
  Figure 1); 2× throughput at 10 KB and 100 KB, none at 1 KB (§3). It runs on
  a userspace TCP stack (XLIO) with NIC TLS offload.
  `../refs/zcl7lb-anrw26.pdf`.
- [bpfbroker] Zhou, Al-Kiswany, Tahmasbi Arashloo, "Toward eBPF-Accelerated
  Pub-Sub Systems", eBPF '25. One `sendto()` per subscriber in the baseline;
  up to 3× throughput at 128 subscribers (§3.1, §3.2). UDP, one-packet
  messages. `../refs/bpfbroker-ebpf25.pdf`.

### Things to keep honest

- Key-value caches are dominated by small values. In Twitter's production
  traces the median of per-cluster mean value sizes is about 230 bytes
  [twitter20]; copy cost is negligible there and per-request cost dominates.
  The claim covers both regimes (the owner, 2026-10-03): small values are
  request-bound, and the eBPF hit path removes the per-request cost; large
  values are copy-bound, and opaque objects remove the per-byte cost.
  Classical KV caches (Redis, Valkey, memcached) are the evaluation's
  backbone; the LLM KV-cache store is a motivating large-value application,
  not the lead (`outline.md`, Framing).
- Sources for the LLM paragraph: [directkv] Luo and Shen, OSDI '26 (quote is
  from its introduction); [cachegen] Liu et al., SIGCOMM '24; [mooncake] Qin
  et al., FAST '25 (best paper); [lmcache] arXiv 2510.09665, unrefereed;
  [rediskv] Redis engineering blog, 2026-03-30, a vendor report and not a
  measurement study. The 2 GB/s figure and chunk sizes come from the blog and
  should be reproduced or replaced by our own measurement. TODO(measure).
- In that blog the copies removed were in the *client*. The server-side claim
  (the store copies every value into and out of socket buffers) is ours to
  measure. TODO(measure).
- [zcl7lb] and [bpfbroker] measure settings unlike SYS's: a userspace stack
  with NIC TLS offload, and one-packet UDP messages. Cite them for the cost,
  not as baselines.
- "Pass-through" is our term. Check that it does not collide with an existing
  one before it goes in the title.
- "Read by many inference nodes that share a prefix" (§2.1) needs a number:
  concurrent readers per block from the Mooncake trace, which carries the
  prefix-block hashes of every request. Prefix-aware routers send
  prefix-sharing requests to the same node, where the hit is local, so the
  remote store sees less concurrency than the sentence implies. TODO(verify)
  from the trace before the sentence stays.
- One model and one chunk size across files. `../LLM.md` sizes with Llama
  3.1 8B (128 KiB per token) and a 32-token chunk; `../experiments/llm_inference.md`
  names Qwen2.5-7B (about 56 KiB per token) and LMCache's default is 256
  tokens. Every size in this section changes by 2× or more with the choice.
- [kanev15]'s 22–27% is a tax that includes much more than copies. Keep it
  behind [libra] and [cai21], which are about the copy itself.
- KV compression. Bytes per token span about 350× across production models
  and the frontier is at the small end (DeepSeek-V4.1-Flash: 890 bytes per
  token in HBM, persistent footprint one eighth of its predecessor). The
  "2 GB/s" and "hundreds of gigabytes" figures are Llama-class; say so. For
  compressed-KV models a store is request-bound, not bandwidth-bound, and
  the saving is memory rather than CPU. Make the claim per chunk size, show
  the break-even, and name which models sit on which side. The same report
  shows the frontier persisting KV to SSD and host memory and trading
  recomputation for storage, which supports the problem's importance and
  the memory contribution. Details and provenance in `../LLM.md` §1; the
  V4.1 numbers are verified against the report in `../refs/`; the V4-Flash
  and V3 per-token figures are from third-party summaries and are
  TODO(verify).
- [dsv41] and [dsv4] are unrefereed technical reports; [dsv3hw] is an arXiv
  paper. Cite them for the model facts only.
