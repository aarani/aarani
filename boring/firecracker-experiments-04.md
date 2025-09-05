# [Let's play with Firecracker](https://github.com/aarani/MangoFaaS) 🧨

## Previously ([Part 3](https://github.com/aarani/aarani/blob/master/boring/firecracker-experiments-03.md)):
We integrated Firecracker microVMs into MangoFaaS to securely run .NET functions for each HTTP request. We optimized storage using OverlayFS, built sparse function images, and compressed them for efficient transfer. The workflow now routes requests through an API Gateway, queues them via Kafka, and launches microVMs on demand—handling everything from image caching to function runtime communication over vsock. Early results show cold starts around 10 seconds, with significant speedup when images are cached or VMs pre-running.

I promised some things in part 3 but I got de-railed and worked on other things (mostly cleanup)

## Let's be real

- That crappy init script I had doesn't take it anymore, it was too basic, it would even panic when dotnet dies, not ideal....
- Creating runtime images was a massive and manual pain in the bottom, not ideal...

## Let's fix that 🧵

After some talking with GH Copilot, I tried compiling systemd for a full day (became 200MB even with its most minimal options), said screw that, and compiled BusyBox init (as a static binary) instead 🤪

I then created a [Dockerfile](https://github.com/aarani/rootfs-generation/blob/main/dotnet/Dockerfile) that builds my rootfs inside docker so I can directly use it for my VMs to make my life easier. 

Turns out "ext4" is a PAIN when used as a readonly disk so instead I switched to SquashFS which is inherently built for being read-only!

# *Tried my solution...* CRASHED!

Turns out BusyBox's init doesn't do anything UNLESS you tell it to in the inittab file! So...... after some serious negotiations with GH Copilot *again* I added mount commands that are required for .NET to work.


```
::sysinit:/bin/mount -t devtmpfs devtmpfs /dev
::sysinit:/bin/mkdir -p /dev/pts
::sysinit:/bin/mount -t devpts devpts /dev/pts
::sysinit:/bin/mount -t proc proc /proc
::sysinit:/bin/mount -t sysfs sys /sys
::sysinit:/bin/mkdir -p /tmp /dev/shm /var/tmp /run
::sysinit:/bin/mount -t tmpfs -o mode=1777,nosuid,nodev tmpfs /tmp
::sysinit:/bin/mount -t tmpfs -o nosuid,nodev tmpfs /dev/shm
::sysinit:/bin/mount -t tmpfs -o mode=0755,nosuid,nodev tmpfs /run
::sysinit:/bin/chmod 1777 /var/tmp

::respawn:/bin/sh -c 'exec socat TCP-LISTEN:55505,fork,reuseaddr VSOCK-CONNECT:2:80'
::once:/bin/sh -c '/bin/dotnet "/app/$(cat /entrypoint.txt)"; rc=$?; echo "dotnet exited rc=$rc, powering off" >&2; sync; poweroff -f'
```

# We're back where we were in Part 3 🤣 but.....

Theoritically, we can convert ANY docker container into runtimes by injecting our init binaries (all static) 🕺

We also had some progress on other sides:
- I'm implementing a SquashFS driver with .NET for no reason. don't ask me why. 🤦🏼‍♂️  Half-way into implementation, I found out I needed ext4 and not SquashFS!
- I added in-process (de)compression (using DEFLATE) instead of that finicky tar command
- Endpoints for adding runtime (Thanks to SquashFS, I don't need to compress runtimes anymore 🕺)

# Roadmap for MangoFaaS
- Binary based communication with runtime
- Dogfood-ing (FaaS based) overlay image creation
- Non-HTTP triggers (Minio events, etc)
- Frontend (no more postman)
- Authroization, Authorization, Authorization 
- Snapshots
- Cleanup terrible Kafka consuming logic, in favour of [kafka-offset-manager](https://github.com/tautvydasversockas/kafka-offset-manager)
