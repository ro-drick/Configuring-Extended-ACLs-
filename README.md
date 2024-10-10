# Packet Tracer Lab: Named Access Control List (ACL) Configuration

## Overview

This Packet Tracer lab demonstrates how to configure **Named Access Control Lists (ACLs)** to enforce security policies on different segments of the network. The objective is to restrict traffic between specific hosts and services as described in the policies below. The network consists of two routers (R1 and R2), switches (SW1, SW2, SW3, SW4), PCs, and servers (SRV1 and SRV2).

### Network Topology

- **Subnets:**
  - **172.16.1.0/24**: Connected to SW1 (Hosts PC1, PC2)
  - **172.16.2.0/24**: Connected to SW2 (Hosts PC3, PC4)
  - **192.168.1.0/24**: Connected to SW3 (Server SRV1)
  - **192.168.2.0/24**: Connected to SW4 (Server SRV2)
  - **203.0.113.0/30**: Serial link between R1 and R2

<img src= "https://github.com/ro-drick/Configuring-Extended-ACLs-/blob/main/extended-acls.jpg">
  
### ACL Policies

1. **Hosts in 172.16.2.0/24 (PC3, PC4) must not communicate with PC1 (172.16.1.1)**.
2. **Hosts in 172.16.1.0/24 (PC1, PC2) must not access the DNS service (UDP Port 53) on SRV1 (192.168.1.100)**.
3. **Hosts in 172.16.2.0/24 (PC3, PC4) must not access HTTP or HTTPS services (Ports 80 and 443) on SRV2 (192.168.2.100)**.

---

## Configuration Steps

### 1. Configure Named ACLs on Router R1

We will create named ACLs on **R1** to implement the required network policies.

#### ACL1: Prevent Hosts in 172.16.2.0/24 from Communicating with PC1

- **Objective:** Block communication from the 172.16.2.0/24 subnet (PC3, PC4) to PC1 (172.16.1.1).

```bash
R1(config)# ip access-list extended BLOCK_PC1
R1(config-ext-nacl)# deny ip 172.16.2.0 0.0.0.255 host 172.16.1.1
R1(config-ext-nacl)# permit ip any any
```

- **Apply to Interface G0/0 (connected to SW1):**

```bash
R1(config)# interface g0/0
R1(config-if)# ip access-group BLOCK_PC1 in
```

#### ACL2: Prevent Hosts in 172.16.1.0/24 from Accessing DNS on SRV1

- **Objective:** Block access to DNS (Port 53) on SRV1 (192.168.1.100) for hosts in 172.16.1.0/24 (PC1, PC2).

```bash
R1(config)# ip access-list extended BLOCK_DNS
R1(config-ext-nacl)# deny udp 172.16.1.0 0.0.0.255 host 192.168.1.100 eq 53
R1(config-ext-nacl)# permit ip any any
```

- **Apply to Interface G0/0:**

```bash
R1(config)# interface g0/0
R1(config-if)# ip access-group BLOCK_DNS in
```

### 2. Configure Named ACLs on Router R2

Next, we configure **R2** for the final policy.

#### ACL3: Prevent Hosts in 172.16.2.0/24 from Accessing HTTP/HTTPS on SRV2

- **Objective:** Block access to HTTP (Port 80) and HTTPS (Port 443) on SRV2 (192.168.2.100) for hosts in 172.16.2.0/24.

```bash
R2(config)# ip access-list extended BLOCK_HTTP_HTTPS
R2(config-ext-nacl)# deny tcp 172.16.2.0 0.0.0.255 host 192.168.2.100 eq 80
R2(config-ext-nacl)# deny tcp 172.16.2.0 0.0.0.255 host 192.168.2.100 eq 443
R2(config-ext-nacl)# permit ip any any
```

- **Apply to Interface G0/1 (connected to SW4):**

```bash
R2(config)# interface g0/1
R2(config-if)# ip access-group BLOCK_HTTP_HTTPS in
```

---

## Verification

### 1. **Verify Communication Block Between 172.16.2.0/24 and PC1:**
- Ping **PC1 (172.16.1.1)** from **PC3 or PC4 (172.16.2.0/24)**. The ping should fail, confirming the ACL is correctly blocking traffic.

```bash
PC3> ping 172.16.1.1
```

### 2. **Verify DNS Block Between 172.16.1.0/24 and SRV1:**
- Attempt to access the **DNS service on SRV1 (192.168.1.100)** from **PC1 or PC2 (172.16.1.0/24)**. The request should be denied.

```bash
PC1> nslookup 192.168.1.100
```

### 3. **Verify HTTP/HTTPS Block Between 172.16.2.0/24 and SRV2:**
- Try accessing **HTTP or HTTPS** services on **SRV2 (192.168.2.100)** from **PC3 or PC4 (172.16.2.0/24)**. Both services should be inaccessible.

```bash
PC3> ping 192.168.2.100
```

---

## Conclusion

This lab demonstrates the use of **Named ACLs** to control access between different segments of the network based on IP addresses and specific services (DNS, HTTP, HTTPS). The configurations successfully implemented the desired network security policies and were verified using ping and service access tests.

---

## Acknowledgements


Special thanks to **Jeremy's IT Lab** for providing valuable resources and tutorials that greatly contributed to the completion of this exercise. His in-depth explanations and practical demonstrations have been instrumental in enhancing my understanding of Cisco networking concepts and the effective use of Packet Tracer.

For more information and additional resources, visit [Jeremy's IT Lab](https://jeremysitlab.com/) and check out his YouTube for the full course, [Jeremy's IT Lab Free CCNA 200-301 | Complete Course](https://www.youtube.com/playlist?list=PLxbwE86jKRgMpuZuLBivzlM8s2Dk5lXBQ)
