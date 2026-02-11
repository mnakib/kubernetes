# Preparing the Prereq

### 1. DNS Records (The Map)

You would need to create these records in your local domain (e.g., `lab.local`).

| Type | Hostname | Points to (IP) | Purpose |
| --- | --- | --- | --- |
| **A** | `cp-vip.lab.local` | `10.0.10.100` | The Load Balancer (HAProxy) IP |
| **A** | `cp-node-01.lab.local` | `10.0.10.11` | Control Plane Server 1 |
| **A** | `cp-node-02.lab.local` | `10.0.10.12` | Control Plane Server 2 |
| **A** | `worker-01.lab.local` | `10.0.10.21` | Worker Server 1 |
| **A** | `worker-02.lab.local` | `10.0.10.22` | Worker Server 2 |

### 2. Install & Configure BIND (DNS Server)

```bash
sudo apt update && sudo apt install bind9 bind9utils bind9-doc -y

```

**Step 2: Configure `named.conf**`
Add your upstream DNS (like Google or your ISP) so the servers can still reach the internet.

```text
forwarders {
    8.8.8.8;
    1.1.1.1;
};

```

**Step 3: Define the Zone in `named.conf.local**`

```text
zone "lab.local" {
    type master;
    file "/etc/bind/db.lab.local";
};

```

**Step 4: Create the Zone File (`/etc/bind/db.lab.local`)**

```text
$TTL    604800
@       IN      SOA     ns1.lab.local. admin.lab.local. (
                              3         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      ns1.lab.local.
ns1     IN      A       10.0.10.10      ; The DNS server itself
cp-vip  IN      A       10.0.10.100     ; HAProxy VIP
cp-node-01 IN   A       10.0.10.11
cp-node-02 IN   A       10.0.10.12
worker-01  IN   A       10.0.10.21

```

### 3. Install & Configure HAProxy (Load Balancer)

This sits on the VIP address (`10.0.10.100`) and passes traffic to the control plane nodes.

**Step 1: Install**

```bash
sudo apt update && sudo apt install haproxy -y

```

**Step 2: Configure `/etc/haproxy/haproxy.config**`
We need to configure "Layer 4" (TCP) load balancing for the Kubernetes API.

```text
frontend k8s-api
    bind *:6443
    mode tcp
    option tcplog
    default_backend k8s-control-planes

backend k8s-control-planes
    mode tcp
    balance roundrobin
    option tcp-check
    server cp-node-01 10.0.10.11:6443 check
    server cp-node-02 10.0.10.12:6443 check

```

**Step 3: Restart and Enable**

```bash
sudo systemctl restart haproxy
sudo systemctl enable haproxy

```
