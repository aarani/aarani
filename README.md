<h1 align="center">Afshin Arani</h1>

<p align="center">
  <em>I build the boring infrastructure that lets other people do interesting things fast.</em>
</p>

<p align="center">
  <a href="https://www.linkedin.com/in/afshinarani/"><img src="https://img.shields.io/badge/LinkedIn-afshinarani-0A66C2?style=flat-square&logo=linkedin&logoColor=white" alt="LinkedIn"></a>
  <a href="mailto:afshin@arani.dev"><img src="https://img.shields.io/badge/Email-afshin%40arani.dev-D14836?style=flat-square&logo=gmail&logoColor=white" alt="Email"></a>
  <a href="https://hpcc.dev"><img src="https://img.shields.io/badge/site-hpcc.dev-0e1014?style=flat-square" alt="hpcc.dev"></a>
  <img src="https://img.shields.io/badge/based%20in-Denmark-C60C30?style=flat-square" alt="Denmark">
</p>

---

```
$ whoami
Senior software engineer / solutions architect. Copenhagen.
Quant engineering at Danske Bank by day: schedulers, EKS, compilers,
certificates, and the kind of systems that get audited.
Protocol implementations in F# and Go by night.

$ cat ~/.interests
distributed systems · sandboxing & isolation · undocumented protocols
compilers & build systems · cryptography · coffee · biotechnology

$ uptime
open source since 2016 — 54 repos and counting
```

---

## 🔭 Currently building

**[hpcc](https://github.com/aarani/hpcc)** — *distributed compilation, but it makes sense* · [hpcc.dev](https://hpcc.dev)

`ccache`, `sccache` and `distcc` all assume the build worker is trusted shared-kernel infrastructure. In a regulated enterprise that assumption ends the conversation. hpcc inverts it: every compile runs inside a per-tenant **Firecracker microVM** (Linux) or **Hyper-V isolated container** (Windows), the worker has no NIC, the container image digest *is* the toolchain identity, and every job lands exactly one audit row.

<sub>`Go` · `Firecracker` · `hcsshim` · `gRPC` · `OIDC` · `Helm` — AGPL-3.0, work in progress, not yet audited.</sub>

**[Craftling](https://github.com/GoCraftling/Craftling)** — *run a fleet of Minecraft servers like cloud infrastructure*

On-demand Minecraft hosting on Firecracker with eBPF-based networking. Yes, it's a toy. No, the networking stack is not.

---

## 🧰 Things I've built

| Project | What it is |
| --- | --- |
| **[NOnion](https://github.com/aarani/NOnion)** | The Tor protocol implemented in pure F# — no interop with the C binaries, so it runs anywhere .NET does. Reading the C source and the spec side by side taught me more cryptography than any course would have. |
| **[DotNetLightning.Kiss](https://github.com/nblockchain/DotNetLightning.Kiss)** + **[geewallet](https://github.com/nblockchain/geewallet)** | Core Lightning Network protocol and wallet logic in F#, including a complete BTC Lightning **Watchtower**. |
| **[TgSharp / TLSharp](https://github.com/nblockchain/TgSharp)** | A Telegram MTProto client in C#. Eight years of chasing an undocumented, moving protocol target. Code generation and reflection got weird, in a good way. |
| **[MangoFaaS](https://github.com/aarani/MangoFaaS)** | Function-as-a-service, from scratch, because I wanted to know how the cold start actually works. |

Also: a **critical severity** vulnerability reported to GitLab via HackerOne. 🐛

---

## 🛠️ Day job, abbreviated

Danske Bank, Copenhagen — building for developers and traders inside a heavily regulated environment

---

## ⚙️ Tools of the trade

<p>
  <img src="https://img.shields.io/badge/C%23-512BD4?style=flat-square&logo=csharp&logoColor=white">
  <img src="https://img.shields.io/badge/F%23-378BBA?style=flat-square&logo=fsharp&logoColor=white">
  <img src="https://img.shields.io/badge/.NET-512BD4?style=flat-square&logo=dotnet&logoColor=white">
  <img src="https://img.shields.io/badge/Go-00ADD8?style=flat-square&logo=go&logoColor=white">
  <img src="https://img.shields.io/badge/C%2FC%2B%2B-00599C?style=flat-square&logo=cplusplus&logoColor=white">
  <img src="https://img.shields.io/badge/TypeScript-3178C6?style=flat-square&logo=typescript&logoColor=white">
</p>
<p>
  <img src="https://img.shields.io/badge/AWS-232F3E?style=flat-square&logo=amazonwebservices&logoColor=white">
  <img src="https://img.shields.io/badge/Kubernetes-326CE5?style=flat-square&logo=kubernetes&logoColor=white">
  <img src="https://img.shields.io/badge/Terraform-7B42BC?style=flat-square&logo=terraform&logoColor=white">
  <img src="https://img.shields.io/badge/Pulumi-8A3391?style=flat-square&logo=pulumi&logoColor=white">
  <img src="https://img.shields.io/badge/Firecracker-FF6B35?style=flat-square&logo=amazonwebservices&logoColor=white">
  <img src="https://img.shields.io/badge/Cilium-F8C517?style=flat-square&logo=cilium&logoColor=black">
  <img src="https://img.shields.io/badge/OpenShift-EE0000?style=flat-square&logo=redhatopenshift&logoColor=white">
</p>

---

<details>
<summary><b>📊 Stats</b></summary>
<br>

<p align="center">
  <img height="165" src="https://github-stats-extended.vercel.app/api?username=aarani&show_icons=true&hide_border=true&include_all_commits=true&theme=transparent">
  <img height="165" src="https://github-stats-extended.vercel.app/api/top-langs/?username=aarani&layout=compact&hide_border=true&langs_count=8&theme=transparent">
</p>

</details>

---

<p align="center">
  <sub>No degree, just commits - self-taught since 2016. Always happy to talk about isolation boundaries, weird protocols, or coffee.</sub>
</p>
