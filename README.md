# 🛡️ Cybersecurity Lab Setup — Kali Linux on VirtualBox

This repository documents my first hands-on cybersecurity lab setup using **Kali Linux on Oracle VirtualBox**, starting from my **Zorin OS** host system.

The goal of this lab was to create a working Kali Linux environment and configure its network so that it can be used for future cybersecurity and networking practice.

---

## 🎯 Objective

The main objective of this lab was to:

* Set up Oracle VirtualBox on Zorin OS
* Download and install a pre-built Kali Linux VirtualBox machine
* Configure the Kali virtual machine
* Create and configure a NAT Network
* Connect Kali to the NAT Network
* Configure the wired network connection
* Troubleshoot a network connection problem
* Verify the network configuration using Linux commands
* Understand some basic networking concepts through hands-on practice

---

## 💻 Host System

The lab was performed on:

* **Host OS:** Zorin OS
* **Virtualization:** Oracle VirtualBox
* **Guest OS:** Kali Linux
* **Architecture:** x86_64

---

# 1. 📥 Getting the Required Software

I started with my Zorin OS system.

For Kali Linux, I used the official Kali Linux website and selected the **pre-built VirtualBox virtual machine** instead of installing Kali from an ISO.

Kali provides pre-built VMware and VirtualBox images specifically for users who want to run Kali as a virtual machine.

### Official Kali Linux source

[Kali Linux — Get Kali](https://www.kali.org/get-kali/?utm_source=chatgpt.com)

### Official VirtualBox source

[Oracle VirtualBox](https://www.virtualbox.org/?utm_source=chatgpt.com)

I also used Oracle VirtualBox as the virtualization software for running Kali Linux.

---

# 2. 📦 Installing 7-Zip on Zorin OS

The Kali VirtualBox image was provided as a compressed `.7z` file.

Since I was using Zorin OS, I installed the 7-Zip package from the terminal.

```bash
sudo apt update
```

Then:

```bash
sudo apt install p7zip-full
```

After installing 7-Zip, I was able to extract the Kali VirtualBox archive.

The Kali documentation also uses `7z` to extract the pre-built VirtualBox image.

---

# 3. 🗜️ Extracting the Kali VirtualBox Image

After downloading the Kali VirtualBox `.7z` file, I opened the terminal and navigated to the directory containing the downloaded file.

I extracted the archive using:

```bash
7z x kali-linux-....7z
```

The extracted files included the VirtualBox machine files needed to start the Kali VM.

The important files were:

```text
.vbox
.vdi
```

The `.vbox` file contains the VirtualBox virtual machine configuration, while the `.vdi` file is the virtual disk used by the virtual machine.

---

# 4. 🖥️ Adding Kali to VirtualBox

After extracting the Kali files, I opened **Oracle VirtualBox**.

Instead of creating a completely new virtual machine manually, I used the existing `.vbox` configuration from the extracted Kali VirtualBox image.

The Kali documentation describes the same process: launch VirtualBox, choose **Add**, and select the extracted `.vbox` file.

After adding the machine, Kali appeared in VirtualBox and was ready to be started.

---

# 5. 🐉 Starting Kali Linux

I started the Kali virtual machine from VirtualBox.

Kali successfully booted and displayed the Kali Linux desktop.

![Kali Linux Running](03-kali-linux-running.png)

At this point, the basic virtual machine installation was complete.

---

# 6. 🌐 Creating a NAT Network

The next step was configuring the network for the cybersecurity lab.

In VirtualBox, I created a **NAT Network** with:

```text
IPv4 Prefix: 10.0.0.0/24
DHCP: Enabled
```

![NAT Network Configuration](01-nat-network-configuration.png)

### Why did I use NAT Network?

During this lab I learned that **NAT** and **NAT Network** are different VirtualBox networking modes.

### NAT

Normal NAT allows a virtual machine to access external networks through the host.

### NAT Network

A NAT Network creates a shared virtual network where multiple virtual machines can be connected to the same network while still being able to access external networks through NAT.

For a future cybersecurity lab containing multiple virtual machines, this makes the NAT Network useful because the machines can communicate with each other within the same virtual network.

---

# 7. ⚙️ Connecting Kali to the NAT Network

After creating the NAT Network, I opened the Kali VM's:

**Settings → Network → Adapter 1**

For **Attached to**, I selected:

```text
NAT Network
```

and selected the NAT Network that I had created.

![Kali Network Adapter](02-kali-network-adapter.png)

This connected the Kali virtual machine to the virtual network.

---

# 8. 🚨 Network Problem Encountered

This was the main problem I encountered during the setup.

After starting Kali, the network connection was not working properly.

I checked the network device status using:

```bash
nmcli device status
```

The `eth0` interface was stuck at:

```text
connecting (getting IP configuration)
```

The output looked like:

```text
DEVICE  TYPE      STATE                                  CONNECTION
eth0    ethernet  connecting (getting IP configuration)  Wired connection 1
lo      loopback  connected (externally)                 lo
```

### What was the problem?

The `eth0` network interface was not completing its IP configuration.

Because of this, Kali remained in the **connecting** state instead of becoming properly connected.

This was an important part of the lab because I had to troubleshoot the problem instead of simply reinstalling Kali.

---

# 9. 🔧 Troubleshooting the Connection

First, I checked the available NetworkManager connections:

```bash
nmcli connection show
```

This showed the connection:

```text
Wired connection 1
```

I then modified the IPv4 DAD timeout for this connection:

```bash
sudo nmcli connection modify "Wired connection 1" ipv4.dad-timeout 0
```

After applying the configuration and restarting the network/Kali as needed, the connection was able to proceed.

### What I learned from this problem

I learned that when a Linux network interface is stuck at:

```text
connecting (getting IP configuration)
```

I should not immediately assume that the entire VM installation is broken.

Instead, I can:

1. Check the network interface.
2. Check the NetworkManager connection.
3. Inspect the current configuration.
4. Make the required changes.
5. Test the connection again.

This was one of the most useful parts of the lab because it gave me actual troubleshooting experience.

---

# 10. 📝 Configuring the Wired Connection

After troubleshooting the connection, I configured **Wired connection 1**.

The IPv4 settings were:

| Setting        | Value      |
| -------------- | ---------- |
| IPv4 Address   | `10.0.0.2` |
| Netmask        | `24`       |
| Gateway        | `10.0.0.1` |
| DNS Server     | `8.8.8.8`  |
| Additional DNS | `10.0.0.1` |

![Wired Connection Configuration](04-wired-connection-configuration.png)

At this stage, I learned the practical purpose of these settings.

### IP Address

`10.0.0.2` identifies the Kali machine on the virtual network.

### Gateway

`10.0.0.1` is the gateway through which Kali can communicate outside its local network.

### DNS

`8.8.8.8` is Google's public DNS server.

The gateway was also configured as a DNS server.

### Netmask

I used `/24` as part of the network configuration. At this stage, I mainly learned how to apply the setting rather than going deeply into subnetting and CIDR calculations.

---

# 11. 🔎 Verifying the IP Configuration

After configuring the connection, I used the Linux command:

```bash
ip addr
```

This displays the available network interfaces and their IP addresses.

![IP Address Verification](05-ip-addr-verification.png)

I used this command to check the network interface and verify the IP configuration of the Kali machine.

---

# 12. 🧠 What I Learned

This lab helped me understand several concepts that I had previously only studied theoretically.

### 🖥️ Virtual Machines

I learned how a complete operating system such as Kali Linux can run inside VirtualBox without replacing my main Zorin OS system.

### 📦 VirtualBox Images

I learned how a pre-built Kali VirtualBox image can be downloaded, extracted, and added directly to VirtualBox.

### 🌐 NAT vs NAT Network

I learned that normal NAT and NAT Network are different.

* **NAT:** mainly provides network access for the VM through the host.
* **NAT Network:** provides network access while allowing multiple VMs connected to the same NAT Network to communicate.

### 📍 IP Address

I learned that a machine needs an IP address to communicate on a network.

In this lab, Kali used:

```text
10.0.0.2
```

### 🚪 Gateway

I learned that the gateway provides a path from the local network to other networks.

In this lab:

```text
10.0.0.1
```

### 📖 DNS

I learned that DNS helps translate domain names into IP addresses.

For example, instead of having to remember an IP address for a website, we can use its domain name.

### 🐧 Linux Networking Commands

I practiced using:

```bash
nmcli device status
```

to check the status of network devices.

I also used:

```bash
nmcli connection show
```

to view NetworkManager connections.

And:

```bash
ip addr
```

to inspect network interfaces and IP addresses.

### 🛠️ Troubleshooting

Most importantly, I learned that networking problems require investigation.

My Kali VM initially remained stuck at:

```text
connecting (getting IP configuration)
```

I used NetworkManager commands to investigate the problem and modified the connection configuration to resolve it.

---

# 13. 📸 Lab Evidence

The `screenshots` folder contains the main evidence from the setup:

1. VirtualBox NAT Network configuration
2. Kali Adapter 1 network configuration
3. Kali Linux successfully running
4. Wired Connection IPv4 configuration
5. IP address verification using `ip addr`

---

# 14. 🚀 Final Result

The Kali Linux virtual machine was successfully installed and configured on my Zorin OS system.

The final setup includes:

```text
Zorin OS
    │
    ▼
VirtualBox
    │
    ▼
Kali Linux
    │
    ▼
NAT Network
    │
    ├── Network: 10.0.0.0/24
    ├── Kali IP: 10.0.0.2
    └── Gateway: 10.0.0.1
```

The lab environment is now ready for future cybersecurity and networking practice.

---

# 🔮 Next Steps

With the basic environment working, I can continue building my cybersecurity skills through:

* Linux practice
* Networking fundamentals
* Network troubleshooting
* Packet analysis
* Security tools
* Cybersecurity labs
* SOC Analyst learning
* Hands-on platforms such as TryHackMe and Hack The Box

---

## ✅ Lab Status

| Task                             | Status      |
| -------------------------------- | ----------- |
| Zorin OS host preparation        | ✅ Completed |
| 7-Zip installation               | ✅ Completed |
| Kali VirtualBox image extraction | ✅ Completed |
| Kali VM added to VirtualBox      | ✅ Completed |
| Kali Linux booted successfully   | ✅ Completed |
| NAT Network configured           | ✅ Completed |
| Kali Adapter configured          | ✅ Completed |
| Network troubleshooting          | ✅ Completed |
| IPv4 configuration               | ✅ Completed |
| IP configuration verified        | ✅ Completed |

---

## 🔗 Official Resources

* [Kali Linux — Get Kali](https://www.kali.org/get-kali/?utm_source=chatgpt.com)
* [Kali Linux VirtualBox Documentation](https://www.kali.org/docs/virtualization/import-premade-virtualbox/?utm_source=chatgpt.com)
* [Oracle VirtualBox](https://www.virtualbox.org/?utm_source=chatgpt.com)

---

**Status: 🟢 Cybersecurity Lab Environment Successfully Set Up**
