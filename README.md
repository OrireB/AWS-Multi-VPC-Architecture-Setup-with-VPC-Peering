# AWS-Multi-VPC-Architecture-Setup-with-VPC-Peering

This project demonstrates how to **create and connect two separate VPCs** using **VPC Peering** to simulate a multi-VPC architecture within AWS.  
You’ll build two isolated networks, each with public and private subnets, then link them together using a peering connection and routing.

---

## 🧠 Key Concepts

### **VPC (Virtual Private Cloud)**
A VPC is a virtual network within AWS that is logically isolated from other networks. You can control who enters and who exits the VPC, as well as manage the resources and traffic within it.

### **Subnet**
A subnet is a range of IP addresses within your VPC that groups related resources (such as EC2 instances).
There are two main types of subnets:
- **Public subnets**: Used for resources that need to connect to or be accessible from the Internet (e.g., web servers).
- **Private subnets**: Used for resources that do **NOT** need direct Internet access (e.g., databases or application servers).

### **CIDR Block (Classless Inter-Domain Routing)**
A CIDR block defines the IP address range for a VPC or subnet.
- Example: 10.0.1.0/24 represents 256 IP addresses.

### **VPC Peering**
A **VPC peering connection** enables two VPCs to communicate with each other privately using their internal IP addresses, as if they were part of the same network.
VPC peering allows network traffic using private IPv4 or IPv6 addresses to flow between VPCs as if they were within the same network.

Peering can occur between:
- VPCs within the same AWS account, or
- VPCs across different accounts (in the same region).

> Note: VPC peering is non-transitive — meaning, if VPC-A is peered with VPC-B and VPC-B is peered with VPC-C, VPC-A cannot automatically communicate with VPC-C.

### **Route Tables**
They define where network traffic from your subnets is directed.  
In this setup, route tables are updated to send traffic to the other VPC via the peering connection.

---

| Concept     | Analogy                                     |
| ----------- | ------------------------------------------- |
| AWS         | Big city                                    |
| VPC         | Private neighborhood                        |
| Subnet      | Street within that neighborhood             |
| CIDR Block  | Address range on that street                |
| Route Table | Traffic rules controlling where vehicles go |

---

## ⚙️ Step-by-Step Setup

### 🧱 Step 1: Create Two VPCs

#### **VPC-A**
1. Go to **VPC → Your VPCs → Create VPC**.  
2. Choose **VPC only**.  
3. Set:
   - **Name tag**: `VPC-A`
   - **IPv4 CIDR block**: `10.10.0.0/16`
4. Leave defaults and click **Create VPC**.

#### **VPC-B**
1. Repeat the same process.  
2. Set:
   - **Name tag**: `VPC-B`
   - **IPv4 CIDR block**: `10.20.0.0/16`

| VPC Name | CIDR Block     |
| -------- | -------------- |
| VPC-A    | `10.10.0.0/16` |
| VPC-B    | `10.20.0.0/16` |


📸 **Screenshot #1:**  
*Both VPCs listed under “Your VPCs” with CIDRs `10.10.0.0/16` and `10.20.0.0/16`.*
![VPC-A](https://raw.githubusercontent.com/OrireB/AWS-Multi-VPC-Architecture-Setup-with-VPC-Peering/22d9f743c86b5129fa8b2c0167b43cda0c7534c0/VPC-A.png)
![VPC-B](https://raw.githubusercontent.com/OrireB/AWS-Multi-VPC-Architecture-Setup-with-VPC-Peering/22d9f743c86b5129fa8b2c0167b43cda0c7534c0/VPC-B.png)

---

### 🌐 Step 2: Create Subnets in Each VPC

#### **For VPC-A**
1. Go to **Subnets → Create subnet**.
2. Select **VPC-A**.
3. Create:
   - **Public Subnet A**: `10.10.1.0/24`
   - **Private Subnet A**: `10.10.2.0/24`

#### **For VPC-B**
1. Repeat for **VPC-B**.
2. Create:
   - **Public Subnet B**: `10.20.1.0/24`
   - **Private Subnet B**: `10.20.2.0/24`

| VPC | Subnet Name | CIDR Block | Type |
|-----|--------------|------------|------|
| VPC-A | Public Subnet A | `10.10.1.0/24` | Public |
| VPC-A | Private Subnet A | `10.10.2.0/24` | Private |
| VPC-B | Public Subnet B | `10.20.1.0/24` | Public |
| VPC-B | Private Subnet B | `10.20.2.0/24` | Private |

> You don’t need to attach Internet or NAT Gateways for this task.

📸 **Screenshot #2:**  
*All 4 subnets with names, VPC association, and CIDR blocks.*
![VPC-A PUBLIC SUBNET.png](https://raw.githubusercontent.com/OrireB/AWS-Multi-VPC-Architecture-Setup-with-VPC-Peering/22d9f743c86b5129fa8b2c0167b43cda0c7534c0/VPC-A%20PUBLIC%20SUBNET.png)
![VPC-A PRIVATE SUBNET.png](https://raw.githubusercontent.com/OrireB/AWS-Multi-VPC-Architecture-Setup-with-VPC-Peering/22d9f743c86b5129fa8b2c0167b43cda0c7534c0/VPC-A%20PRIVATE%20SUBNET.png)
![VPC-B PUBLIC SUBNET.png](https://raw.githubusercontent.com/OrireB/AWS-Multi-VPC-Architecture-Setup-with-VPC-Peering/22d9f743c86b5129fa8b2c0167b43cda0c7534c0/VPC-B%20PUBLIC%20SUBNET.png)
![VPC-B PRIVATE SUBNET.png](https://raw.githubusercontent.com/OrireB/AWS-Multi-VPC-Architecture-Setup-with-VPC-Peering/22d9f743c86b5129fa8b2c0167b43cda0c7534c0/VPC-B%20PRIVATE%20SUBNET.png)

---

### 🔗 Step 3: Create a VPC Peering Connection

1. Go to **VPC → Peering Connections → Create peering connection**.
2. Configure:
   - **Name**: `VPC-A-and-VPC-B`
   - **Requester VPC**: `VPC-A`
   - **Accepter VPC**: `VPC-B`
3. Click **Create peering connection**.
4. After creation, select it → **Actions → Accept request**.

📸 **Screenshot #3:**  
*The VPC Peering Connection in **Active** state.*
![VPC-A and VPC-B PEERING](https://raw.githubusercontent.com/OrireB/AWS-Multi-VPC-Architecture-Setup-with-VPC-Peering/22d9f743c86b5129fa8b2c0167b43cda0c7534c0/VPC-A%20and%20VPC-B%20PEERING.png)

---

### 🗺️ Step 4: Update Route Tables

#### **In VPC-A**
1. Go to **Route Tables → find the main route table for VPC-A**.
2. Click **Edit routes → Add route**:
   - **Destination**: `10.20.0.0/16`
   - **Target**: The Peering Connection ID created.
3. Save changes.

#### **In VPC-B**
1. Do the same for **VPC-B**’s main route table.
2. Add route:
   - **Destination**: `10.10.0.0/16`
   - **Target**: The same Peering Connection ID.
3. Save changes.

📸 **Screenshot #4:**  
*Both route tables with routes pointing to the other VPC via the peering connection.*
![VPC-A ROUTE TABLE.png](https://raw.githubusercontent.com/OrireB/AWS-Multi-VPC-Architecture-Setup-with-VPC-Peering/22d9f743c86b5129fa8b2c0167b43cda0c7534c0/VPC-A%20ROUTE%20TABLE.png)
![VPC-B ROUTE TABLE.png](https://raw.githubusercontent.com/OrireB/AWS-Multi-VPC-Architecture-Setup-with-VPC-Peering/22d9f743c86b5129fa8b2c0167b43cda0c7534c0/VPC-B%20ROUTE%20TABLE.png)

---

## Understanding the Route Table Entries
When updating route tables, the **destination** is the CIDR block of the other VPC.
- **VPC-A Route Table:**
“If you need to reach any IP in `10.20.0.0/16`, send traffic via the peering connection.”

_ **VPC-B Route Table:**
“If you need to reach any IP in `10.10.0.0/16`, send traffic via the peering connection.”

Think of it like setting up a road between two neighborhoods:
- VPC-A’s map says: “To reach VPC-B, take the Peering Connection Highway.”
- VPC-B’s map says: “To reach VPC-A, take the Peering Connection Highway.”
This ensures seamless communication between the two VPCs.

---

## Final Architecture Overview
- **Two VPCs:**
  - VPC-A → `10.10.0.0/16`
  - VPC-B → `10.20.0.0/16
- **Each VPC:**
  - 1 Public Subnet
  - 1 Private Subnet
- **A Peering Connection:** Established between VPC-A and VPC-B
- **Route Tables:** Updated in both VPCs to allow bidirectional communication through the peering connection

---

## 🧪 Verification — How to Test Connectivity

### 1. Ping Test
- Launch EC2 instances in both VPCs (private subnets recommended).
- Allow **ICMP traffic** in both security groups.
- From the instance in VPC-A, ping the private IP of the instance in VPC-B (and vice versa).
- From the EC2 instance in VPC-A, run:
  - ping 10.20.1.10
- From the EC2 instance in VPC-B, run:
  - ping 10.10.1.10
- If the ping succeeds, the peering connection is working.

### 2. SSH Test
- Allow **SSH (port 22)** in security groups.
- Attempt to SSH from one instance to the other using private IP addresses.

### 3. Route Table Check
- Confirm that the routes exist and point to the peering connection.
- The target for these routes should be the peering connection ID.

### 4. Peering Connection Status
- Go to **VPC → Peering Connections** and verify the status is **Active**.

### 5. Flow Logs (Optional)
- Enable **VPC Flow Logs** to monitor packet traffic between the two VPCs for troubleshooting.

If the ping or SSH test succeeds, your VPC peering connection is working correctly.

---

## 👨‍💻 Author
**Orire Bankole**  
Cloud / DevOps Enthusiast 
🌐 [Twitter or X Portfolio Link](https://x.com/_Lorisann)
