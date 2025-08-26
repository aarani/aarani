# [Let's play with Firecracker](https://github.com/aarani/MangoFaaS) 🧨

## Previously ([Part 1](https://github.com/aarani/aarani/blob/master/boring/firecracker-experiments-01.md)):

I explored AWS Firecracker — a lightweight VMM behind Lambda & Fargate — compared it with containers, chose to build a snapshot-based mini-FaaS, and got my first Firecracker microVM running (with custom rootfs + prebuilt kernel) on a Linux VM. After fixing networking, I confirmed it works and started planning to manage VMs programmatically from C#.

# Preparation

As mentioned in the previous part, I now have a simple PoC API Gateway that matches HTTP requests to Function Id + Function Version using a route table stored in Database with in-memory caching.

# Function Id? Function Version? Wat?!

*You are making a good point!* I still don't have a way to create functions.

## What is even a function?

I asked copilot:
> *A function in FaaS is a small, stateless, event-triggered piece of code executed on-demand in a managed, ephemeral runtime that scales per invocation.*

To run this "piece of code" I need to create an image, which we described how to create in the previous part.

For this part, we have a new problem to tackle, as soon as I tried to run too many microVMs on my not-so-micro VM, I ran into space issues.

How can my not-so-cheap VM keep up...

The answer was not-so-many google searches away!

# OverlayFS

In a nutshell, OverlayFS is a feature of Linux Kernel where it combines multiple different underlying mount points into one, resulting in a single directory structure that contains underlying files and sub-directories from all sources.

## How is that helpful?

So, one of the most common applications of OverlayFS is overlaying a read/write partition over a read-only partition. What that means? 

Essentially we can save disk space by sharing the same read-only rootfs between all VMs.

## How do they persist data then?

**That's the beauty of OverlayFS**, you can easily overlay a separate read-write disk on top of your read-only disk where VMs can read/write from/to.

<img width="784" height="344" alt="image" src="https://github.com/user-attachments/assets/1a2a56ea-b53f-4e5d-826b-9b0fb5d88d6b" />

## Talk is cheap, show me the code!!!

There are many resources on how to do overlayfs on the web so let's add one more resource.

1. Create your base rootfs image.
2. Open it and create the following folders inside it
   `mkdir -p /mnt/rootfs/overlay/root /mnt/rootfs/overlay/work /mnt/rootfs/mnt /mnt/rootfs/rom`
3. Copy [this](https://github.com/firecracker-microvm/firecracker-containerd/blob/f05e3ff/tools/image-builder/files_debootstrap/sbin/overlay-init) init file *This is a separate init file that runs before your main init file and does the prepration for you* (you can remove the line that creates the volumes folder, we don't need it)
4. Time for the dear old chmod +x
5. umount out and you're done!

## What about the overlay image? How can I create that?

1. Create a sparse file using dd:
   `dd if=/dev/zero of=overlay.ext4 conv=sparse bs=1M count=1024`
3. `mkfs.ext4 overlay.ext4`
4. That's it! You're done!


As I'm writing this I'm also working on the C# implementation of a simple FaaS platform which you can see [here](https://github.com/aarani/mangofaas).

<img width="1654" height="1107" alt="image" src="https://github.com/user-attachments/assets/a0efb973-b6ef-4af9-b20e-0e61aea3517f" />


Sources: 
  - https://github.com/aarani/mangofaas
  - https://github.com/njapke/overlayfs-in-firecracker/blob/2de8bf8/README.md
  - https://en.wikipedia.org/wiki/OverlayFS
  - https://github.com/firecracker-microvm/firecracker-containerd/blob/f05e3ff/tools/image-builder/files_debootstrap/sbin/overlay-init
