# Azure VM Networking Lab: VNets, Subnets, and Network Security Groups

### Overview
This lab focused on designing and validating a tiered network architecture in Microsoft Azure using Virtual Networks (VNets), subnets, and Network Security Groups (NSGs). Two Linux virtual machines were deployed into separate subnets to simulate a web tier and an application tier. Network access was intentionally restricted and tested to demonstrate how Azure enforces traffic flow and security boundaries at the subnet level.

All virtual machines were managed as headless servers using SSH, reflecting standard cloud and system administration practices.

---

### Objectives
- Design a virtual network with multiple subnets
- Deploy virtual machines into separate network tiers
- Enforce traffic flow using Network Security Groups
- Validate allowed and denied traffic paths
- Implement a jump-host access pattern
- Troubleshoot and confirm network behavior

---

### Architecture
- One Azure Virtual Network (`vnet-prod-lab`)
- Two subnets:
  - `subnet-web` (public-facing web tier)
  - `subnet-app` (private application tier)
- Two Linux virtual machines:
  - `vm-web-01` with a public IP
  - `vm-app-01` with no public IP
- Two Network Security Groups:
  - `nsg-web` associated with `subnet-web`
  - `nsg-app` associated with `subnet-app`
- SSH used for administrative access
- Nginx used to validate HTTP connectivity

---

### Technologies Used
- Microsoft Azure
- Azure Virtual Network
- Azure Network Security Groups
- Azure Virtual Machines
- Ubuntu Server 22.04 LTS
- SSH
- Nginx

---

## Step 1 — Create a Resource Group

- Resource group name: rg-vm-networking-lab

<img width="512" height="288" alt="image" src="https://github.com/user-attachments/assets/465bcaff-031f-4c62-b7f7-a24a27f85641" />

## Step 2 — Create the Virtual Network

- Go to Virtual networks → Create

- Name: vnet-prod-lab

- Address space: 10.0.0.0/16

- Create two subnets:

  - subnet-web → 10.0.1.0/24

  - subnet-app → 10.0.2.0/24

<img width="512" height="288" alt="image" src="https://github.com/user-attachments/assets/e755f963-fb8d-47f5-aeb9-f414fa5bd99b" />

## Step 3 — Create Network Security Groups

Create two NSGs:

NSG 1: Web NSG

- Name: nsg-web

- Inbound rules:

  - Allow SSH (22) from your IP

  - Allow HTTP (80) from Internet

- Outbound: default rules are fine

NSG 2: App NSG

- Name: nsg-app

- Inbound rules:

  - Allow SSH only from 10.0.1.0/24

  - Deny all inbound from Internet (implicit deny)

- Outbound: default rules

<img width="512" height="288" alt="image" src="https://github.com/user-attachments/assets/871c0a10-75e2-4427-9fac-308ac516c2e8" />
<img width="512" height="288" alt="image" src="https://github.com/user-attachments/assets/88e815db-fcea-4f85-8a12-086c632f4bfc" />

## Step 4 — Associate NSGs with Subnets

- Associate nsg-web → subnet-web

- Associate nsg-app → subnet-app

## Step 5 — Deploy Web VM (Public)

- VM name: vm-web-01

- Subnet: subnet-web

- Public IP: Enabled

- Authentication: SSH key

- OS: Ubuntu Server

- Size: pick a small VM to keep costs low

 <img width="512" height="288" alt="image" src="https://github.com/user-attachments/assets/9e42fb8f-2b8c-43cf-bc4a-569e822417ed" />

## Step 6 — Deploy App VM (Private)

- VM name: vm-app-01

- Subnet: subnet-app

- Public IP: Disabled (this is super important to make sure that this is private and doesn't have a public facing IP)

- Authentication: SSH key

- OS: Ubuntu Server

## Step 7 — Install Services

- On both VMs run these commands to install nginx

```
sudo apt update
sudo apt install -y nginx
```

## Step 8 — Test Traffic Flow

Test 1: Internet → Web VM

- Browser: http://<web-vm-public-ip>

- Expected: SUCCESS

<img width="512" height="288" alt="image" src="https://github.com/user-attachments/assets/46a3f629-5fc5-4447-b542-b41f818a3944" />

Test 2: Internet → App VM

- Try to SSH or browse directly

- Expected: FAIL

- Reason: no public IP + NSG deny

<img width="512" height="288" alt="image" src="https://github.com/user-attachments/assets/121e40a2-be32-4570-8f13-6ba3cead539b" />
<img width="512" height="288" alt="image" src="https://github.com/user-attachments/assets/c72225a7-54dc-4a40-8284-90b90af6d212" />

Test 3: Web VM → App VM

- From vm-web-01:

- curl http://10.0.2.4

- Expected: SUCCESS

<img width="512" height="288" alt="image" src="https://github.com/user-attachments/assets/f673b35b-dcae-4bfa-82e1-8b6b26d0f855" />

## Step 9 — Cleanup

- Delete resource group rg-vm-networking-lab

---

## Results
- Public traffic successfully reached the web tier.
- The application tier remained private and inaccessible from the internet.
- Internal communication between tiers functioned only when explicitly allowed by NSG rules.

---

## Conclusion and Takeaways
This lab demonstrated how Azure networking components work together to enforce secure, tiered access within a virtual network. By separating workloads into dedicated subnets and applying Network Security Groups at the subnet level, traffic flow was tightly controlled and validated through deliberate testing. The use of a jump-host access pattern ensured that private resources were not exposed to the public internet while remaining accessible to trusted internal systems. I leanred alot and there were some hiccups in the setting up and assignment of NSG. During the testing of the web VM, the nginx page wasn't loading like expected. I was about to trace back to a misassignment of the web VM to the app NSG which didn't allow incoming traffic from the internet. From there I was able to rectify this and get the rest of the lab to work as expected. Overall, this lab provided practical experience designing, securing, and validating Azure network architectures aligned with real-world cloud environments.
