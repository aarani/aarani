# Let's play with Firecracker 🧨

## What is AWS Firecracker?
Firecracker is an open source virtual machine monitor from AWS that runs lightweight microVMs on KVM, optimized for secure multi tenancy, fast startup, and low overhead. It powers AWS Lambda and Fargate by providing strong isolation like VMs with startup times closer to containers.

## What's the difference between Firecracker and a simple docker container?

- Firecracker uses KVM hardware virtualization with its own guest kernel; containers share the host kernel. Firecracker provides stronger multi‑tenant isolation.
- Containers typically start in tens of ms; Firecracker cold starts are ~100–300 ms (You can optimize the heck out of them so they can start in single-digit ms🤯)
- While microVMs are lightweight, they still have their own guest kernel so density is lower in comparison to containers.
- Containers use OCI images, firecracker runs VM images (kernel + rootfs) so you need to create your own images (You can use things like firecracker-containerd but I decided to start from scratch)

## What Copilot told me to do?

> ME: I want to build something using firecracker
>
> Copilot: Option (A): Firecracker Launchpad CLI Goal: A single binary/CLI that takes a declarative YAML (vcpu, mem, kernel, rootfs, networking, vsock mappings) and brings up / tears down Firecracker microVMs.
> 
> Copilot: Option (B): Snapshot-Accelerated “Mini FaaS” Goal: Accept short-lived function code (Python handler), pre-warm runtime, take a snapshot, and serve HTTP invocations by restoring microVMs quickly.
> 
> Copilot: Option (C): Hardened Multi-Tenant Code Runner Goal: Accept untrusted user code (e.g., C or Python), compile/run inside isolated Firecracker microVMs with strict seccomp & read-only base image; return stdout/stderr & resource usage.

A was too easy, 🥱

C reminded me a lot of those auto-judge applications for ACM programming contests while fun, it was too niche, 🤓

### Let's go for B! 🧨🕺

### API Gateway PoC
I didn't really stick to the project plan though, instead I started a WebAPI project that takes all requests (all paths, hosts, methods) and forwards them to Kafka.

### Firecracker comes into play!
I have a Mac and I can't really use firecracker on it so I bought a Linux VM from DigitalOcean (Hetzner doesn't support nested virtualization) and I now need rootfs + kernel so I tried building linux kernel (a shallow clone of it),
took too long on my cheap VM, I got bored so let's just use Amazon prebuilt ones!

```
ARCH="$(uname -m)"
release_url="https://github.com/firecracker-microvm/firecracker/releases"
latest_version=$(basename $(curl -fsSLI -o /dev/null -w  %{url_effective} ${release_url}/latest))
CI_VERSION=${latest_version%.*}
latest_kernel_key=$(curl "http://spec.ccfc.min.s3.amazonaws.com/?prefix=firecracker-ci/$CI_VERSION/$ARCH/vmlinux-&list-type=2" \
    | grep -oP "(?<=<Key>)(firecracker-ci/$CI_VERSION/$ARCH/vmlinux-[0-9]+\.[0-9]+\.[0-9]{1,3})(?=</Key>)" \
    | sort -V | tail -1)

# Download a linux kernel binary
wget "https://s3.amazonaws.com/spec.ccfc.min/${latest_kernel_key}"

```

This gave me the linux kernel, ROOTFS TIME!

```
docker pull mcr.microsoft.com/dotnet/aspnet:9.0
CONTAINER_ID=$(docker create mcr.microsoft.com/dotnet/aspnet:9.0)
docker export $CONTAINER_ID -o rootfs.tar
dd if=/dev/zero of=rootfs.ext4 bs=1M count=2048
mkfs.ext4 rootfs.ext4
mkdir /mnt/rootfs
sudo mount -o loop rootfs.ext4 /mnt/rootfs
sudo tar -xf rootfs.tar -C /mnt/rootfs
```

This gave me a rootfs image, but there's a twist! 
Containers don't really use init processes so many contianer images do not have them! After some serious talks with Copilot, looking around google, I created a sample init file and moved on!

I used the quick start guide [here](https://github.com/firecracker-microvm/firecracker/blob/main/docs/getting-started.md#getting-a-firecracker-binary), to start a VM. 

## It works!!! 😍
## It doesn't have internet access 😭

Both container images I tried did not have the "ip" command installed, [I used this small note in the guide to setup the network](https://github.com/firecracker-microvm/firecracker/blob/main/docs/network-setup.md#advanced-guest-network-configuration-using-kernel-command-line).
I also added 1.1.1.1 as my nameserver and... 
## YAAAY! It really works this time.

## I found out that each firecracker process should be used for one tenant so I now need to run firecracker binary from C#, communicate with it and boot a VM from C#....

## I drew a simple map which I'll revise throughout my experiment...

![Untitled-2025-03-23-1658](https://github.com/user-attachments/assets/2d93bbd9-6599-46e2-8f87-da2aeda2eaa1)
