# [Let's play with Firecracker](https://github.com/aarani/MangoFaaS) 🧨

## Previously ([Part 2](https://github.com/aarani/aarani/blob/master/boring/firecracker-experiments-02.md)):

- Added an API Gateway mapping HTTP routes to (FunctionId, FunctionVersion), but the function creation workflow is still missing.
- Ran into disk bloat when launching many Firecracker microVMs each with its own full rootfs.
- Adopted OverlayFS: one shared read-only base rootfs + per‑VM sparse writable ext4 layer.
- Simple setup: prep base rootfs with overlay dirs + custom overlay init; create per‑VM sparse overlay (dd + mkfs.ext4).

## *Enough Preparation!* Let’s start breaking down [MangoFaaS](https://github.com/aarani/MangoFaaS)!

**I’m an HTTP request**. My journey starts in MangoFaaS.Gateway. I carry multiple identifiers—most notably *Host* and *Path*. MangoFaaS.Gateway uses these to determine the tenant (owner) and which function version to invoke.

**I’m a function owner**. My journey starts in MangoFaaS.Functions. I register a new function by uploading a ZIP of my .NET application to S3 along with metadata: *Entrypoint* and *Runtime*.

MangoFaaS.Functions runs a background job that discovers new function version ZIPs in a bucket, builds a function overlay image, and pushes it to an images bucket.

It’s worth noting: images are [sparse files](https://en.wikipedia.org/wiki/Sparse_file). To avoid transferring all the zero‑filled regions, we compress them (currently with GNU tar because .NET’s Tar implementation doesn’t preserve sparse file sparsity). So we shell out to the “lovely” GNU tar.

**I’m the aforementioned HTTP request.** I’ve been routed and I now sit in Kafka. I move toward the first free MangoFaaS.Firecracker.Node (future improvement: prefer nodes that already have my function cached).

**I’m MangoFaaS.Firecracker.Node.** I:
- Put the request in a pending list
- Download (if not already cached) the latest kernel, the runtime base image, and the function’s overlay image
- Launch (or prepare) a Firecracker microVM and wait for the function runtime to connect to my Kestrel web server over vsock and request work

**I’m the function runtime to be executed.**
- I connect and poll the Kestrel server over vsock for HTTP requests
- I process the request and send the HTTP response back over the same vsock channel

That’s it—the response travels back through MangoFaaS.Firecracker.Node and returns via the MangoFaaS.Gateway node that originally handled the inbound request.

<img width="453" height="193" alt="image" src="https://github.com/user-attachments/assets/54072e49-4a4a-46f0-a3e4-655a1da29d67" />

Early timing:
- Very cold start (image downloads required): ~10 s
- Not‑so‑cold (images already cached locally): ~6 s
- Warm (VM already running): negligible

<img width="1851" height="204" alt="image" src="https://github.com/user-attachments/assets/8c4bb305-2bbe-42f2-a9cf-df6d95e139d3" />

Next: snapshots, prewarming, smarter scheduling, cache affinity.
