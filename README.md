<img src="https://cdn.prod.website-files.com/677c400686e724409a5a7409/6790ad949cf622dc8dcd9fe4_nextwork-logo-leather.svg" alt="NextWork" width="300" />

# Launching VPC Resources

**Project Link:** [View Project](https://nextwork.ai/projects/8f49d756-9e51-58c0-856a-e71e54f064ab)

**Author:** Maria Jasmin Ahorro  
**Email:** jasminahorro23@gmail.com

---

![Image](https://nextwork.ai/overjoyed_magenta_proud_yak/uploads/8f49d756-9e51-58c0-856a-e71e54f064ab_8ee57662)

## Introducing Today's Project!

### What is Amazon VPC?

Amazon VPC (Virtual Private Cloud) is a logically isolated, custom virtual network dedicated to your AWS account, and it is useful because it gives you complete control over your cloud environment—allowing you to define IP address ranges, configure subnets, route tables, and network gateways, and enforce granular security controls to safely isolate and run your AWS resources.

### How I used Amazon VPC in this project

I used Amazon VPC to build an isolated virtual network, launch public and private EC2 instances in separate subnets, and configure custom security groups and route tables to safely manage network traffic and protect private resources.

### One thing I didn't expect in this project was...

One thing I didn't expect in this project was how fast and seamless creating an entire network architecture could be using the Amazon VPC wizard's "VPC and more" feature with the visual resource map, which automatically configured subnets, route tables, and gateways all from a single screen instead of having to build each component manually from scratch.

### This project took me...

This project took me 20 minutes to complete, during which I configured EC2 instances in both public and private subnets, set up dedicated security groups, and used the Amazon VPC wizard with resource maps to automate custom network creation.

## Setting Up Direct VM Access

Directly accessing a virtual machine means logging into and managing its operating system or software remotely over the network, as if you were sitting right in front of the physical machine.

While tools like the AWS Management Console allow you to manage cloud infrastructure at a high level, direct access lets you perform OS-level administrative tasks such as installing software, editing system configuration files, running custom scripts, or running diagnostic commands directly through protocols like SSH.

### SSH is a key method for directly accessing a VM

SSH traffic means encrypted data exchanged over network port 22 using the Secure Shell protocol to establish a secure connection between your local computer and a remote virtual machine, such as an EC2 instance. This type of traffic carries administrative commands, terminal inputs, and system responses, ensuring that sensitive data and credentials cannot be intercepted or read by unauthorized users over the network.

### To enable direct access, I set up key pairs

Key pairs are a set of secure cryptographic credentials consisting of a public key and a private key, used to authenticate identity and grant secure access to virtual machines like AWS EC2 instances. The public key is stored directly on the AWS instance, while the private key file (such as a .pem file) is saved locally on your computer. When you initiate a connection over SSH, the virtual machine uses its public key to generate an encrypted challenge that can only be decrypted by your corresponding private key. This key-based authentication eliminates the need for traditional passwords, protecting remote connections against brute-force attacks and ensuring safe administrative access to your servers.

A private key's file format determines how the cryptographic key data is encoded and structured so specific operating systems, servers, and software applications can read it. My private key's file format was .pem (Privacy Enhanced Mail), which is a widely supported, plain-text container format commonly used by AWS EC2 instances and Linux environments for SSH authentication.

## Launching a public server

I had to change my EC2 instance's networking settings by opening the Network settings panel during instance creation, selecting my custom NextWork VPC instead of the default VPC, choosing the NextWork Public Subnet, and attaching the existing NextWork Public Security Group to ensure the instance was placed in the correct network environment with the intended firewall rules.

![Image](https://nextwork.ai/overjoyed_magenta_proud_yak/uploads/8f49d756-9e51-58c0-856a-e71e54f064ab_88727bef)

## Launching a private server

My private server has its own dedicated security group because it requires stricter inbound access controls than the public server. Instead of opening SSH traffic to any IP address on the internet (0.0.0.0/0), its dedicated security group restricts SSH access specifically to resources within the NextWork Public Security Group, isolating the private server from public exposure while allowing secure administrative communication from within the VPC.

My private server's security group's source is set to the NextWork Public Security Group, which means only instances or resources assigned to that specific public security group can initiate SSH connections to the private server. This isolates the private server from direct internet access, ensuring that administrative traffic can only originate from trusted internal resources within the VPC.

![Image](https://nextwork.ai/overjoyed_magenta_proud_yak/uploads/8f49d756-9e51-58c0-856a-e71e54f064ab_4a9e8014)

## Speeding up VPC creation

I used an alternative way to set up an Amazon VPC! This time, I selected the VPC and more option in the Amazon VPC creation wizard, which allowed me to automatically provision my entire network architecture—including subnets, route tables, and an internet gateway—all from a single page using the interactive VPC resource map.

A VPC resource map is a visual diagram inside the AWS Management Console that displays the architectural layout of your Virtual Private Cloud, showing how your subnets, route tables, network connections, and internet gateways are connected and interact with each other at a glance.

My new VPC has a CIDR block of 10.0.0.0/16. It is possible for my new VPC to have the same IPv4 CIDR block as my existing VPC because AWS VPCs are logically isolated virtual networks within your account and region. Because they are completely separate by default, IP address conflicts will not occur unless you attempt to establish direct inter-VPC communication, such as through VPC peering.

![Image](https://nextwork.ai/overjoyed_magenta_proud_yak/uploads/8f49d756-9e51-58c0-856a-e71e54f064ab_1cbb1b88)

## Speeding up VPC creation

### Tips for using the VPC resource map

When determining the number of public subnets in my VPC, I only had two options: 0 or 2. This was because AWS follows high availability best practices by default. When allocating public subnets across 2 Availability Zones, the wizard automatically provisions a public subnet in each zone to ensure redundancy, while keeping the configuration simple and preventing single points of failure.

The set up page also offered to create NAT gateways, which are managed AWS devices that enable instances in a private subnet to connect to the internet or other AWS services for outbound traffic (such as downloading software updates or security patches) while preventing the public internet from initiating inbound connections directly to those private instances.

![Image](https://nextwork.ai/overjoyed_magenta_proud_yak/uploads/8f49d756-9e51-58c0-856a-e71e54f064ab_8ee57662)

---

## Notes & corrections

- **How SSH key pairs really work:** modern SSH doesn't encrypt a challenge with the public key. The client *signs* data with the **private key**, and the server *verifies* the signature with the **public key** stored on the instance. The private key never leaves your computer either way.
- **Why only 0 or 2 public subnets:** the "VPC and more" wizard creates the same number of public subnets in **every** Availability Zone you choose. With 2 AZs, the count has to be 0 or a multiple of 2. The wizard's layout follows high-availability practice, but the AZ count is the real reason for the choice.
- **Same CIDR block:** two VPCs with overlapping CIDRs (both `10.0.0.0/16`) can exist side by side, but they **can't be peered**. AWS rejects peering connections between overlapping ranges. Plan unique ranges if the VPCs might ever need to talk to each other.
- **NAT gateway cost:** NAT gateways bill per hour plus per GB processed, even when idle. Pick "None" in the wizard unless you need one, and delete any you create when you finish the project.
- **Clean up:** terminate both EC2 instances and delete the extra VPC so nothing keeps billing. Store the `.pem` key file somewhere safe and never commit it to Git.
- The last screenshot links to the same image as the cover at the top. It may have been meant to show the wizard's subnet/NAT options.

---

*Built with [NextWork](https://nextwork.ai) - [View this project](https://nextwork.ai/projects/8f49d756-9e51-58c0-856a-e71e54f064ab)*
