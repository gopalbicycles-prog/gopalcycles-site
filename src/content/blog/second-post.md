---
title: 'Setting up free GPS tracking for a small rental fleet on Oracle Cloud'
description: 'Oracle Cloud Free Tier gives you a always-free VM that is powerful enough to self-host Traccar for fleets under 25 devices. Here is the full setup walkthrough.'
pubDate: 'May 15 2025'
heroImage: '../../assets/blog-placeholder-2.jpg'
---

If you run a small rental fleet — say 5 to 25 e-bikes or scooters — paying $10–30 per device per month to a fleet management SaaS adds up fast. Oracle Cloud's Always Free Tier gives you a VM with 1 OCPU and 1 GB RAM that runs Traccar comfortably for small fleets at zero cost.

## What you need

- Oracle Cloud account (free, no credit card required for Always Free resources)
- A domain or subdomain pointed at your VM IP (via Cloudflare)
- 30–60 minutes

## Steps

**1. Provision an Always Free VM**

In the Oracle Cloud console, create a Compute instance using the Always Free shape (`VM.Standard.E2.1.Micro`). Choose Ubuntu 22.04. Note the public IP.

**2. Install Traccar**

```bash
wget https://www.traccar.org/download/traccar-linux-64.zip
unzip traccar-linux-64.zip
sudo ./traccar.run
sudo systemctl enable traccar
sudo systemctl start traccar
```

Traccar listens on port 8082 by default.

**3. Set up Cloudflare Tunnel (recommended)**

Rather than opening ports directly, use a Cloudflare Tunnel to expose your Traccar instance at `traccar.yourdomain.com`. This keeps your VM's IP private and adds DDoS protection for free.

**4. Connect your devices**

Add your devices in Traccar using their IMEI or identifier. Most GPS-enabled IoT hardware (Teltonika, Omni, Segway, etc.) supports the Traccar protocol or a compatible variant.

**5. Wire up GopalDesk**

Point GopalDesk at your Traccar hostname. The Cloudflare Worker handles auth so your credentials stay server-side.

For fleets under 25 devices this setup runs entirely on free infrastructure.
