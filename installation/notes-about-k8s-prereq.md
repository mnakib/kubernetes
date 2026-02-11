# Notes about the Prereq

Think of Kubernetes like a high-end apartment complex for your containerized applications, and to build this complex apartment, you need to lay the foundation; the plumbing (network), the lobby security (load balancer), and the street signs (DNS). Without these, the "residents" (your apps) won't be able to talk to each other or receive guests.

Here is a breakdown of what we need to get started.

---

## 1. Network: The Internal Infrastructure

In Kubernetes, everything needs an IP address. We don't just have one IP per server; every individual application "container" gets its own.

### What I need from you:

We need three distinct IP address ranges (subnets). Don't worry, **only the Node Network needs to be routable on your existing physical switches.**

* **Node Network (Physical/VM):** This is the standard network where the actual servers live. These need static IPs and must be able to talk to each other and the internet (for downloading updates).
* **Pod Network (Internal):** This is a large, private "virtual" range used for container-to-container talk.
* **Service Network (Internal):** A separate private range used for internal load balancing between groups of containers.

> **Why we need this:** Kubernetes treats every application like its own tiny server. If we don’t have these separate lanes, the traffic gets congested, and the apps can't find one another.

---

## 2. Load Balancer: The Front Door

Since our applications can move from one server to another if a hardware failure occurs, we can’t give users a single server's IP address.

### What I need from you:

* **A Virtual IP (VIP):** I need a single entry point that sits in front of my "Control Plane" nodes.
* **External Access:** Eventually, we’ll need a way to map external web traffic (Port 80/443) to the cluster.

> **Why we need this:** The Load Balancer acts like a smart receptionist. If one server goes down, the Load Balancer automatically sends the "guest" (the user) to a server that is still working. It ensures the system stays highly available.

---

## 3. DNS: The Map

Computers love numbers, but humans (and our apps) love names.

### What I need from you:

* **Node Records:** Standard A-records for the physical servers.
* **API Record:** A specific name (e.g., `k8s-cluster.company.com`) pointing to that Load Balancer VIP we discussed.
* **Upstream Forwarding:** The cluster will have its own internal DNS, but it needs to be able to "ask" your corporate DNS servers how to find things like our internal databases or the internet.

> **Why we need this:** If "App A" needs to talk to "App B," it shouldn't have to memorize a random IP address that might change. DNS allows our services to find each other by name, making the whole system much more resilient.

---

## Summary Checklist for the Team

| Component | Requirement | Note |
| --- | --- | --- |
| **Nodes** | Static IPs on a dedicated VLAN | The physical foundation. |
| **Pod Subnet** | A large /16 or /24 range (non-routable) | Only exists "inside" Kubernetes. |
| **Service Subnet** | A large /16 or /24 range (non-routable) | Only exists "inside" Kubernetes. |
| **Load Balancer** | 1 VIP + Health Checks | Points to the Master nodes. |
| **DNS** | Internal resolution + API endpoint | Helps us find the "front door." |

---

### Cluster Prerequisites Worksheet

Here is a technical specification template.

| Component | Technical Requirement | Recommended Range | Actual Value (To be filled) |
| --- | --- | --- | --- |
| **Node Network** | Standard Routable VLAN | e.g., `10.0.10.0/24` |  |
| **Pod Network** | Virtual Overlay (Non-routable) | e.g., `192.168.0.0/16` |  |
| **Service Network** | Virtual Overlay (Non-routable) | e.g., `10.96.0.0/12` |  |
| **Control Plane VIP** | Single Static IP for Load Balancer | From Node Network |  |
| **DNS Server(s)** | Your existing Upstream DNS | e.g., `10.0.0.2`, `10.0.0.3` |  |


## 4. Firewall Requirements

**For the Network Admin:**

* **Port Openings:** We’ll need ports **6443** (API Server), **2379-2380** (Database sync), and **10250** (Kubelet) open between the nodes.
* **Internet Access:** The nodes need access to `registry.k8s.io` and `docker.io` to pull the system images.

**For the Sysadmin / DNS Admin:**

* **Hostname Resolution:** Please ensure each node’s hostname (e.g., `k8s-node-01`) is resolvable via your DNS.
* **The "API" Record:** Create a record like `cluster-api.yourdomain.com` that points to the **Control Plane VIP** provided by the network team.
