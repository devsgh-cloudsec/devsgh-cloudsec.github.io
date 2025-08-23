---
title: "🔐 How I Built a Secure Homelab with Proxmox & pfSense (Without Breaking My Home Network)"
date: 2025-04-05 10:00:00 +0000
categories: [Networking, Homelab]
tags: [pfSense, Proxmox, VLAN, Firewall, Jekyll, Virtualization]
pin: true
math: true
mermaid: true
image:
  path: /assets/img/homelab-network-diagram.png
  alt: Network Diagram showing Proxmox, pfSense, and isolated lab network
---

> **💡 TL;DR:** I wanted to build a safe homelab to practice networking with **Proxmox** and **pfSense** — without risking my main home network. This is how I did it, step by step, with full network isolation, routing, and shared management access. Let me show you how you can do the same. 🛠️

---

## 🧩 The Problem: I Didn’t Want to Break the Internet (Again) 😅

We’ve all been there. You’re excited. You’ve got a spare PC, a fresh install of Proxmox, and a burning desire to play with firewalls, VLANs, and virtual networks.

But then reality hits.

> “What if I mess up my home network? What if my kids lose Wi-Fi during dinner because I accidentally nuked the DHCP server?” 🍝

Yeah. That happened to me. Once. 😬

So this time, I decided: **No more risks.** I wanted a lab — isolated, secure, and connected only when I wanted it to be.

Enter: **Proxmox + pfSense in an isolated network**, bridged safely to my main network. 🔗

And guess what? It works *beautifully*. ✨

---

## 🎯 My Goal: The Homelab Dream 🌟

- Use a **dedicated PC** for Proxmox ⚙️
- Run **pfSense as a VM** inside Proxmox 💻
- Create a **completely isolated lab network**: `192.168.50.0/24` 🔒
- Let the lab **get internet** via pfSense through my main network (`192.168.1.0/24`) 🌐
- Allow **management access** to both Proxmox and pfSense on the **same port** 🖥️
- **Zero interference** with my existing setup 🙅‍♂️

**Mission:** Safe learning. ✅

---

## 🗺️ Network Map: The Big Picture 🧭

Here’s how everything fits together:

```mermaid
graph TD
    A[Internet] --> B[pfSense - Main Router<br>192.168.1.1]
    B --> C[Main Network<br>192.168.1.0/24]
    C --> D[Proxmox Host<br>192.168.1.100]
    D --> E[pfSense VM]
    E --> F[Lab Network<br>192.168.50.0/24]
    F --> G[Lab Devices<br>192.168.50.10-254]
    E -- WAN --> H[192.168.1.101]
    E -- LAN --> I[192.168.50.1]
    D -- Management --> I
