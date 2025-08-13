---
title: Networking
linktitle: 🌐Networking
type: book
date: "2021-11-08T00:00:00+01:00"
tags:
  - VirtualBox
  - Vagrant

# Prev/next pager order (if `docs_section_pager` enabled in `params.toml`)
weight: 1
---

<!--more-->

🖧 VirtualBox offers powerful networking features that let your virtual machines (VMs) interact with the outside world🌍 or stay completely isolated🔒. 

🏡 Our Reference Setup:

Before diving into the networking modes, here’s the layout for illustration:
- 🌐 Internet connects to a Router (192.168.1.1)
- 🖥️ Desktop (wired) — runs VirtualBox with 3 VMs
- 💻 Laptop (wireless) — part of the same home network
Inside VirtualBox on the desktop, we’ve got three VMs running any OS you like—Linux, Windows, BSD, etc. This setup helps us explore how each networking mode behaves.
![](/images/uploads/virtualbox-networking.png)

## NAT

⬆️ One-Way Internet Access

Network Address Translation (NAT) creates a private tunnel between the host and a single VM.
- 🖥️ VM can access the internet via the host’s network card
- 🚫 VM is invisible to other devices (laptop, other VMs)

Think of NAT as a one-way street: the VM can go out, but nothing comes in unless you open the gates.
![](/images/uploads/virtualbox-nat.png)

## NAT Network

⬆️⬆️ NAT Network: A Virtual LAN with Internet
NAT Network expands on NAT by allowing multiple VMs to share a virtual network.
- 🖥️ VMs can talk to each other freely
- 🌐 All VMs have internet access

```VirtualBox``` acts like a mini-router, assigning IPs and routing traffic—just like your home router does for your desktop and laptop.

![](/images/uploads/virtualbox-natnetwork.png)

## Bridge Network

⬆️⬇️ Bridged Networking: Full LAN Integration

Bridged mode puts your VMs directly on your physical network.
- 📡 VMs get IPs from your router (e.g., 192.168.1.x)
- 🔄 VMs, host, and laptop can all communicate
- 🧪 Perfect for server testing, peer-to-peer apps, and full network visibility

It’s perfect for testing servers, peer-to-peer apps, or anything that needs full network visibility.

![](/images/uploads/virtualbox-bridge.png)

## Host-Only Network

🛡️ Host-Only: Isolated but Connected to Host
Host-Only networking creates a private network between the host and its VMs.
- 🖥️ Host ↔ VMs: ✅ Communication allowed
- 🔄 VMs ↔ VMs: ✅ Communication allowed
- 🌐 VMs ↔ Internet: ❌ No access
- 💻 VMs ↔ Laptop: ❌ No access
Great for secure development environments or internal simulations without touching your external network.

![](/images/uploads/virtualbox-hostonly.png)

## Internal Network

🕸️ Internal Network: VM-Only Communication

The three virtual machines will be able to see each other they will either need to be set to static IP addresses or else one of them will need to act as a DHCP server however this network is unable to communicate with the host

- VMs communicate only with each other.
- Host and external network are completely cut off.
- You define the network name to group VMs together.
🧪 Best for: Simulating isolated networks, testing firewalls, or building sandboxed clusters.

![](/images/uploads/virtualbox-internal-network.png)

## Summary

| Mode            | VM → Internet | Internet → VM | VM ↔ Host | VM ↔ VM | Use Case |
|-----------------|----------------|----------------|-----------|--------|----------|
| **NAT**         | ✅ Yes         | ❌ No          | ❌ No      | ❌ No   | Simple outbound Internet; isolated VM |
| **NAT Network** | ✅ Yes         | ❌ No*         | ✅ Yes     | ✅ Yes  | Multi-VM setup with outbound Internet and host access |
| **Bridged**     | ✅ Yes         | ✅ Yes         | ✅ Yes     | ✅ Yes  | VM acts like a LAN device; full visibility |
| **Host-Only**   | ❌ No          | ❌ No          | ✅ Yes     | ✅ Yes  | Isolated testing; no Internet, but host access |
| **Internal**    | ❌ No          | ❌ No          | ❌ No      | ✅ Yes  | Fully isolated VM cluster; no host or Internet |

> **Note**: Internet → VM in NAT Network is blocked by default. You can enable it via **manual port forwarding**.