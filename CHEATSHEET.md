# Cheat Sheet: Launching VPC Resources

Lesson notes from the NextWork project, for quick review.

## Contents

1. [Security groups vs network ACLs](#1-security-groups-vs-network-acls)
2. [Launching an EC2 instance: AMI, instance type, tenancy](#2-launching-an-ec2-instance-ami-instance-type-tenancy)
3. [Key pairs & direct VM access](#3-key-pairs--direct-vm-access)
4. [SSH](#4-ssh)
5. [Reading an instance's networking details](#5-reading-an-instances-networking-details)
6. [Public vs private server security](#6-public-vs-private-server-security)
7. [The "VPC and more" wizard & resource map](#7-the-vpc-and-more-wizard--resource-map)
8. [CIDR blocks: VPCs, subnets, sizes, IPv6](#8-cidr-blocks-vpcs-subnets-sizes-ipv6)
9. [NAT gateways vs internet gateways](#9-nat-gateways-vs-internet-gateways)
10. [VPC endpoints](#10-vpc-endpoints)
11. [DNS hostnames & resolution](#11-dns-hostnames--resolution)
12. [Analogy table](#12-analogy-table)
13. [Self-quiz](#13-self-quiz)

---

## 1. Security groups vs network ACLs

| | Inbound rules | Outbound rules |
|---|---|---|
| Controls | Traffic **entering** your resources | Traffic your resources **send out** |
| Examples | Website visitors, form submissions | Server calls another service, sends an email |

**Network ACLs** also use inbound and outbound rules, but they work at the **subnet** level.
- **Rule 100** (default NACL) allows **all** inbound and outbound traffic.
- The **`*` rule** is a catch-all deny for traffic that matches no numbered rule. It never fires here because rule 100 already allows everything.
- Rules are checked **lowest number first**, and the first match wins.

| | Security group | Network ACL |
|---|---|---|
| Applies to | Instance (network interface) | Subnet |
| Stateful? | **Yes**: reply traffic is allowed automatically | **No**: return traffic needs its own rule |
| Rule types | Allow only | Allow **and** deny |
| Default | New SG: no inbound, all outbound | Default NACL: allow all. A **custom** NACL denies all until you add rules |

> The stateful/stateless row and the custom NACL default aren't from the lessons. They're standard AWS facts worth knowing for exams.

---

## 2. Launching an EC2 instance: AMI, instance type, tenancy

- **AMI (Amazon Machine Image):** the *software* template. It holds the OS and preinstalled apps. Like buying a computer with Windows already set up.
- **Instance type:** the *hardware*, meaning CPU, memory, storage and network performance (e.g. `t2.micro`).
- **Tenancy:**
  - **Default (shared):** runs on hardware shared with other AWS customers. Cheap, and the standard choice.
  - **Dedicated:** hardware used only by you. Used for compliance-heavy work such as healthcare. Costs more.

**Networking settings at launch:** pick your **VPC** (not the default), the **subnet**, and the **security group**.

---

## 3. Key pairs & direct VM access

- **Directly accessing a VM** means logging into its OS remotely as if you were sitting at it. You need this to install software, edit config files, run scripts and troubleshoot. The console manages *infrastructure*; SSH gives you the *OS*.
- **Key pair** = a **public key** (stored on the instance) + a **private key** (kept by you).
- **Key pair type:** **RSA** (Rivest-Shamir-Adleman) is common and widely supported. AWS also offers **ED25519**, which is newer and shorter, but Windows AMIs don't support it.
- **Private key file format:**
  - **`.pem`** (Privacy Enhanced Mail) is used by OpenSSH, Linux, macOS and modern Windows.
  - **`.ppk`** is used by PuTTY on Windows.

> **Correction:** the lesson says the server encrypts a challenge with the public key. Modern SSH works the other way round: your client **signs** with the private key, and the server **verifies** the signature with the public key. Either way, the private key never leaves your machine.

> **Never** commit a `.pem` to Git. On Linux/macOS, run `chmod 400 key.pem` or SSH will refuse it as "too open."

---

## 4. SSH

- **SSH (Secure Shell)** is the protocol for securely logging into a remote machine. It uses **TCP port 22**.
- SSH checks that you hold the private key that matches the public key on the server.
- Once connected, **everything is encrypted**: commands, output and credentials.
- SSH is still the standard way in. Teams use IaC and automation to *reduce* manual SSH, but it's still used for live troubleshooting, manual updates and odd config changes.

```bash
ssh -i nextwork-keypair.pem ec2-user@<public-ip>
```

---

## 5. Reading an instance's networking details

| Field | Meaning |
|---|---|
| **Availability Zone** | The specific data center area within the Region where the instance runs |
| **VPC ID** | Which VPC the instance belongs to (e.g. NextWork VPC) |
| **Subnet** | The IP range the instance's private IP comes from. A public subnet has a route to an **internet gateway** |
| **Public IPv4 address** | A globally unique address that makes the instance reachable from the internet |

> A normal public IP **changes when you stop/start** the instance. Use an **Elastic IP** if you need a fixed one.

---

## 6. Public vs private server security

| | Public server SG | Private server SG |
|---|---|---|
| SSH source | `0.0.0.0/0` (anywhere), or better, *your IP only* | **The public server's security group** |
| Who can SSH in | Anyone with the key | Only resources in the public SG |

- Using a **security group as the source** means "allow traffic from anything wearing that SG." No IP addresses to maintain.
- This is the basis of a **bastion host / jump box** pattern: SSH into the public server, then hop to the private one.

---

## 7. The "VPC and more" wizard & resource map

- **"VPC only"** creates just the VPC, and you build everything else by hand (as in the earlier project).
- **"VPC and more"** creates the VPC, subnets, route tables, internet gateway, and optionally NAT gateways and endpoints, all on one page.
- **Resource map:** a live diagram of how subnets, route tables and gateways connect. It's much easier than reading lists, especially for big VPCs.
- **Name tag auto-generation** tags every resource from one name you enter (e.g. `NextWork-subnet-public1-us-east-1a`).
- **Public subnet count: 0 or 2 only (with 2 AZs).** The wizard puts one public subnet in **each** AZ for **redundancy** and **high availability**, so one AZ going down doesn't take you offline.
  - You *can* get a single public subnet by choosing **1 AZ**.
  - You need more? Add them by hand later. The default limit is **200 subnets per VPC**.

---

## 8. CIDR blocks: VPCs, subnets, sizes, IPv6

- **Two VPCs can share a CIDR block** (both `10.0.0.0/16`) because VPCs are isolated. But overlapping VPCs **can't be peered**, so the best practice is a **unique CIDR per VPC**.
- **Subnets inside one VPC can't overlap.** They share one network, so overlapping ranges would make routing impossible.

| Prefix | IP addresses | Typical use |
|---|---|---|
| `/8` | 16,777,216 | Huge networks (not subnets) |
| `/16` | 65,536 | VPCs (also the largest VPC AWS allows) |
| `/20` | 4,096 | Wizard's default subnet size, a middle ground |
| `/24` | 256 | Small subnets |
| `/28` | 16 | Smallest subnet/VPC AWS allows |
| `/32` | 1 | One single IP (e.g. "my IP" in an SG rule) |

> **Shortcut:** addresses = 2^(32 − prefix). Every +1 on the prefix halves the size.
> **AWS reserves 5 IPs in every subnet**, so a `/24` gives 251 usable addresses and a `/20` gives 4,091.

- **IPv6:** a far bigger address space (~340 undecillion, versus IPv4's ~4.3 billion). You can skip it unless your app or users need it.

---

## 9. NAT gateways vs internet gateways

| | Internet gateway | NAT gateway |
|---|---|---|
| Used by | Public subnets | Private subnets |
| Direction | **Both ways** (in + out) | **Outbound only**; blocks unrequested inbound |
| Instance needs public IP? | **Yes** | **No**: the NAT translates to its own public IP |
| Cost | Free | **Hourly + per-GB**, even when idle |
| Lives in | Attached to the VPC | A **public** subnet, with an Elastic IP |

**Why not just an internet gateway with no inbound rules?** Instances would still need public IPs, and a public IP means more attack surface. One misconfigured SG and the instance is exposed. A NAT gateway keeps private instances private.

> **Tip:** set NAT gateways to **None** in the wizard for practice projects, or delete them right after.

---

## 10. VPC endpoints

- Some services like **S3 live outside your VPC**, so by default your traffic to them goes over the public internet.
- A **VPC endpoint** connects your VPC to an AWS service **privately**, staying on the AWS network. That improves security and can cut data costs.
- **S3 gateway endpoint** is the most common one, and it's what the wizard offers. **Gateway endpoints (S3, DynamoDB) are free.**
- Other services use **interface endpoints** (PrivateLink), which cost money.

---

## 11. DNS hostnames & resolution

- **DNS hostnames** give instances readable names instead of only numeric IPs. Real formats look like `ec2-3-80-1-2.compute-1.amazonaws.com` (public) or `ip-10-0-1-5.ec2.internal` (private).
- **DNS resolution** lets AWS's DNS server translate those names into IPs.
- **Why it matters:** IPs can change, but a name stays consistent, so references keep working.
- Both should be **on**. Many services, including interface endpoints, need them.

---

## 12. Analogy table

| Concept | Analogy |
|---|---|
| VPC | Your own gated neighborhood in AWS |
| Subnet | A street in the neighborhood |
| Internet gateway | The neighborhood's main gate, open both ways |
| NAT gateway | A mail drop-off: residents can send letters out and get replies, but strangers can't knock on their door |
| Security group | A bouncer at each house's door, who remembers who went out (stateful) |
| Network ACL | A guard at the street entrance, who checks every car both ways (stateless) |
| AMI | A computer that comes with the OS already installed |
| Instance type | The computer's specs: CPU, RAM |
| Key pair | A lock on the server (public key) + the only key that opens it (private key) |
| SSH | A secure phone line to the server's terminal |
| Resource map | The neighborhood map at the entrance |
| VPC endpoint | A private tunnel to S3 instead of driving on the public highway |
| DNS hostname | A contact name in your phone instead of a raw number |
| Dedicated tenancy | Renting a whole house instead of an apartment |

---

## 13. Self-quiz

<details><summary>1. What's the difference between inbound and outbound rules?</summary>
Inbound rules control traffic coming <b>into</b> your resources. Outbound rules control traffic your resources <b>send out</b>.
</details>

<details><summary>2. What does the <code>*</code> rule in a network ACL do?</summary>
It's a catch-all <b>deny</b> for traffic that matched no numbered rule.
</details>

<details><summary>3. Security group or network ACL: which is stateless?</summary>
The network ACL. Return traffic needs its own rule.
</details>

<details><summary>4. AMI vs instance type?</summary>
AMI = software and OS template. Instance type = hardware (CPU, memory, etc.).
</details>

<details><summary>5. Where do the public and private keys live?</summary>
The public key is on the EC2 instance. The private key (.pem) is on your computer.
</details>

<details><summary>6. What port does SSH use?</summary>
TCP 22.
</details>

<details><summary>7. Why is the private server's SSH source set to the public security group?</summary>
So only resources in the public SG can SSH in. The private server isn't reachable from the internet.
</details>

<details><summary>8. Can two VPCs have the same CIDR block? What's the catch?</summary>
Yes, because they're isolated. But they can't be peered, so use unique CIDRs as best practice.
</details>

<details><summary>9. Why can't subnets in the same VPC overlap?</summary>
They're in the same network, so overlapping ranges cause IP conflicts and traffic can't be routed.
</details>

<details><summary>10. How many IPs are in a /20? How many can you use in AWS?</summary>
4,096 in total. 4,091 are usable, because AWS reserves 5 per subnet.
</details>

<details><summary>11. Why does the wizard offer only 0 or 2 public subnets?</summary>
With 2 AZs, it puts one public subnet in each AZ for redundancy and high availability.
</details>

<details><summary>12. NAT gateway vs internet gateway?</summary>
An IGW allows traffic both ways and needs instances with public IPs. A NAT gateway allows outbound only for private instances with no public IP.
</details>

<details><summary>13. Why use a NAT gateway instead of an IGW with no inbound rules?</summary>
An IGW still needs public IPs on the instances, which increases the attack surface. A NAT gateway keeps them private.
</details>

<details><summary>14. What is a VPC endpoint, and which one is most common?</summary>
A private connection from your VPC to an AWS service that doesn't use the public internet. The S3 gateway endpoint is the most common.
</details>

<details><summary>15. DNS hostnames vs DNS resolution?</summary>
Hostnames give instances readable names. Resolution translates those names into IP addresses.
</details>

<details><summary>16. Default vs dedicated tenancy?</summary>
Default runs on shared hardware and is cheap. Dedicated runs on hardware for you alone, costs more, and is used for compliance.
</details>
