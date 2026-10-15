---
title: Simulating a PX4 quadcopter over the internet
date: 2026-10-15 17:40:00 +0200
categories: [Projects]
tags: [quadcopter, simulation, px4, project log]
---

# Flying a Simulated Drone Over the Internet: Gazebo + PX4 + Tailscale

Running a drone simulation is easy when everything lives on one laptop. It gets more interesting when you want the heavy lifting — Gazebo, PX4 SITL, GPU-accelerated rendering — to happen on a beefy remote server, while you fly and watch the video feed from a laptop on a completely different network. This post walks through a simple setup using **Tailscale** for networking and a **prebuilt headless Gazebo + PX4 Docker image**, so you don't have to compile anything.

## Why Tailscale

Gazebo, PX4, and QGroundControl all expect to talk to each other over specific ports (UDP for MAVLink, RTSP for video), and normally that means dealing with port forwarding, NAT traversal, or VPN configuration. Tailscale sidesteps all of that: install it on both machines, and each gets a stable, private IP that's routable between them as if they were on the same LAN - no firewall rules to open.

![Block Diagram of the links between entities](/assets/img/simulated-drone-mermaid-diagram.png)

## Step 1: Connect both machines to Tailscale

Install Tailscale on your GPU server and on your Mac, and log both into the same tailnet:

```bash
# On both the server and your Mac
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up
```

Once connected, grab your Mac's Tailscale IP — you'll need it in the next step:

```bash
tailscale ip -4
```

## Step 2: Run the headless Gazebo + PX4 image on the server

On the GPU server, launch the simulation using [jonasvautherin's px4-gazebo-headless image](https://github.com/JonasVautherin/px4-gazebo-headless):

```bash
docker run --rm -it -p 8554:8554 \
  jonasvautherin/px4-gazebo-headless:1.17.0 \
  -v gz_x500_mono_cam -w baylands \
  100.77.195.14 100.77.195.14
```

A quick breakdown of this command:

- `-p 8554:8554` exposes the RTSP video port from the container to the server's host network
- `-v gz_x500_mono_cam` selects the vehicle model — here, an X500 quadrotor with a mono camera
- `-w baylands` selects the Gazebo world
- The two trailing `100.77.195.14` arguments are your **Mac's** Tailscale IP, telling PX4 SITL where to stream MAVLink traffic (the address is repeated because the image expects separate arguments for the SITL and MAVLink Router targets)

At this point the server is simulating the drone and physics headlessly, streaming video over RTSP and MAVLink over the tailnet toward your Mac.

## Step 3: Watch the video feed with VLC

On your Mac, open the RTSP stream coming from the server:

```bash
vlc rtsp://<server-tailscale-ip>:8554/live
```

Swap in the server's Tailscale IP (from `tailscale ip -4` run on the server). If VLC hangs on connect, it's often a transport negotiation issue — try forcing RTSP over TCP:

```bash
vlc --rtsp-tcp rtsp://<server-tailscale-ip>:8554/live
```

## Step 4: Control the drone with QGroundControl

![Simulated quadcopter being controlled over tailscale using QGroundControl](/assets/img/simulated_drone_control_tailscale.png)

Open QGroundControl on your Mac. Because the container was told to send MAVLink to your Mac's Tailscale IP in Step 2, QGroundControl should auto-detect the SITL instance and connect without any manual configuration — no need to add a comm link by hand.

From here you have full control: arm, take off, fly waypoints, and watch the live camera feed in VLC alongside your telemetry in QGC.

## Wrapping up

The whole pipeline breaks down to: Tailscale removes the networking friction, the prebuilt Docker image removes the build friction, and RTSP/MAVLink do the rest. This is a handy pattern any time you want simulation compute on a remote GPU box but want to fly and observe from a laptop — including over different networks or even different countries, since it's all just tailnet traffic.
