---
title: PXE-based provisioning server with FOG Project & netboot.xyz integration
date: 2025-06-14 00:00:00
categories: [OS,Linux,Deployment]
tags: [linux,terminal,os]
---
## 🎯 Goal:
We're building a powerful PXE-based provisioning server that offers both:

 - Standardised OS image deployment using FOG Project (ideal for captured Windows, Linux, or macOS "golden" templates).

 - On-demand installer and rescue tools via netboot.xyz (boot live environments, installers, recovery utilities).

Think of this as having your own network-based OS autopilot—redeploy full images or test boot utilities from a single interface.

### 🛠 Tools We'll Use

#### 1. <a href="https://fogproject.org/" target="_blank"> FOG Project </a>

 - A free, open-source, Linux-based network imaging and management system.

 - Supports efficient capture/deployment (via Partclone), BIOS & UEFI boot, multicast, snap-ins, remote tasks, and printer/antivirus management netboot.xyz.

 - Boots clients entirely over the network using PXE/TFTP/iPXE—no installation media required.


#### 2. <a href="https://netboot.xyz/" target="_blank"> netboot.xyz </a>
 - A prebuilt iPXE menu that allows unified, network-based selection and boot of installers, live sessions, and rescue tools (Linux distros, Windows PE, recovery utilities).

 - Lightweight (under 1 MB), auto-updating, and can be self-hosted via Docker or directly via HTTP/TFTP.

### Why Combine Them?
 - FOG Project: perfect for fast, repeatable deployments of your pre-configured images.

 - netboot.xyz: ideal for on-the-fly booting of OS installers, troubleshooting tools, or live environments.

 - Integrating them gives you a single PXE menu where users can choose between deploying a golden image or booting a utility—boosting flexibility without extra complexity.

## 🎛️ Server Hardware / VM Requirements
### 🖥️ Operating System
 - Linux-compatible: Ubuntu 18+, Debian 11+, CentOS/RHEL 7+, Fedora 22+ 
 - netboot.xyz is light. You can run it on a 64-bit Linux (x86_64 or arm64) via Docker.

### 🧠 CPU & RAM
Based on a small-to-medium deployment scenario:
 - **Minimum:** 2 vCPU + 4 GB RAM – enough for basic operation in Linux GUI/text mode.
 - **Recommended for up to ~200 clients:** 4 vCPU + 6–8 GB RAM to manage client check-ins, database processes, and Docker overhead.

### 💾 Storage
 - **Root OS disk:** 32 GB (standard Linux install).
 - **Images directory (**`/images`**):** SSD or fast RAID array, size according to your images (e.g., 500 GB – 2 TB).
 - **Optional snap-ins/apps storage (**`/opt/fog/`**):** separate partition for flexibility.

### 🌐 Networking
To function correctly, your PXE server environment must meet these networking needs:

#### DHCP Server
 - Required to tell PXE clients where to get their bootloader.
 - (Optional) Fog can provide DHCP or your existing DHCP service (e.g., router or dedicated DHCP server).
 - DHCP must point to the PXE server's IP and specify the boot file (e.g., undionly.kpxe for BIOS or ipxe.efi for UEFI).

 - Configure DHCP options in your DHCP Server:
 
 | Option | Name | Description | Example Value |
| ------ | -------------------- | ------------------------------------ |------------------------------------- |
| 66 | **TFTP Server Name** | The IP address of your FOG server    | `x.x.x.x` |
| 67 | **Boot File Name**   | The bootloader file to load via TFTP | Bootloader: undionly.kpxe (BIOS) or ipxe.efi (UEFI) |

> Optional: Configure your dnsmasq to answer PXE/iPXE requests and give the proper file. We’ll make a tutorial for this later.
{: .prompt-tip}

#### TFTP Server
 - Used to deliver the PXE/iPXE bootloader to the client.
 - Provided by the FOG server.
 
#### HTTP Server
 - Used to serve files like images, bootloaders, and iPXE scripts.
 - FOG includes Apache for this. netboot.xyz's menu is loaded via HTTP directly from the internet (boot.netboot.xyz).

#### Bandwidth
 - **Minimum:** 1 Gbps NIC (enough for small environments).
 - **Recommended:** 10 Gbps or LACP (2×1 Gbps) for large-scale or fast deployments.

>***Note:*** netboot.xyz does not act as a DHCP server. It expects your DHCP and PXE environment to be in place. FOG covers this perfectly.
{: .prompt-info}


### ⚙️ Server Roles
This setup combines two systems: FOG and `netboot.xyz`. Here's who does what:

| Component            | Role                                    | Provided By                     |
| -------------------- | --------------------------------------- | ------------------------------- |
| **DHCP Server**      | Assigns IP to clients and points to PXE | FOG (optional) or external DHCP |
| **TFTP Server**      | Delivers PXE/iPXE bootloader            | FOG |
| **PXE Boot Menu**    | Displays image deployment & tools menu  | FOG |
| **iPXE Chainloader** | Boots dynamic tools like netboot.xyz    | iPXE entry in FOG menu |
| **HTTP File Host**   | Hosts images and bootable resources     | FOG (Apache) + netboot.xyz CDN |

>With this setup, F**OG handles all core PXE infrastructure, while netboot.xyz is added as an optional iPXE menu entry**. Providing a dynamic, always up-to-date toolkit.
{: .prompt-info}

### ✅ Requirements Summary Table

| Component          | Spec                                   | Notes                                 |
| ------------------ | -------------------------------------- | ------------------------------------- |
| **CPU**            | 2 vCPU (min), 4 vCPU (recommended)     | Handles FOG + Docker/control tasks    |
| **RAM**            | 4 GB (min), 6–8 GB (recommended)       | For services and Docker overhead      |
| **Storage**        | 32 GB root + SSD(s) for images         | Fast disk for image capture/deploy    |
| **Network**        | 1 Gbps NIC (10 Gbps or LACP for scale) | Essential for high-throughput imaging |
| **OS**             | Ubuntu/Debian/CentOS/Fedora (64-bit)   | Supports FOG and netboot.xyz     |
| **Docker support** | Yes – for netboot.xyz container        | Hosted alongside FOG services         |



## 1. 📦 Installing and Configuring the FOG Server

> For this tutorial, it's assumed that you already have an OS configured and ready to go. We are using a Debian with 4 GB of RAM and a disk of 32 GB. Disk setup is a volume group, so in the future, we can increase its space if required.
{: .prompt-info}

### 🛠 Step 1 – Install Required Packages
```bash
sudo apt update
sudo apt install -y git curl
```

### 📥 Step 2 – Download FOG Installer

Clone the official FOG Project repository:

```bash
git clone https://github.com/FOGProject/fogproject.git
cd fogproject/bin
```

### ⚙️ Step 3 – Run the FOG installer

Launch the installer:

```bash
sudo ./installfog.sh
```

You'll now enter an interactive installation wizard. Follow these prompts:

#### 🧩 Installer Prompts & Recommended Answers
> **Pro Tip:** These are just suggestions; read the prompt carefully and answer according to your reality.
{: .prompt-tip}

| Prompt| Suggested Input|
| ------------------------------- | -------------------------------------------------------------------------------------------------------------------------- |
| **What version of Linux would you like to run the installation for?**  | `2` for **Debian** Installation |
| **What type of installation?**  | `N` for Normal Installation |

| **Interface to use?**           | Choose the correct network interface (e.g., `eth0` or `ens18`)|

| **Would you like to setup a router address for the DHCP server** | Select `Y` |
| **What is the IP address to be used for the router on** | Place the IP of your Router |
| **Would you like DHCP to handle DNS?** | Select `Y` |
| **What DNS address should DHCP allow?** | Use the IP of your internal DNS |
| **Use FOG as the DHCP server?** | `Y` if you don’t have another DHCP on your network <br> `N` if another service (e.g., router or Windows Server) handles it |
| **would you like to install the additional language packs?** | Proceed accordingly |
| **Would you like to enable secure HTTPS on your FOG server?** | Select `N` |
| **Share information** | Select `Y` |
| **Are you sure you wish to continue** | Select `Y` |
| **Installation continues…** | It will install Apache, PHP, MySQL/MariaDB, TFTP, NFS, and other dependencies. |

> Complete **Step 4** before pressing enter!
{: .prompt-warning}

### 🌐 Step 4 – Complete Web-Based Setup

Once the terminal installer finishes, it will prompt:
``"Go to http://<YOUR_IP>/fog/management in your browser to finish the setup."``

1. Open a browser and navigate to:
2. ``http://<your-server-ip>/fog/management``
3. Click “Install/Update Now”
4. Wait for confirmation that the FOG database schema has been installed/updated.
5. Go back to the CLI and press `Enter`.
6. Done! Now you can go to the FOG login page.

#### 📝 Default credentials:

***Username:*** `fog`

***Password:*** `password`

### 🔐 Step 5 – Update FOG Password (Recommended)

After the first login:

1. Go to Users → List All Users → fog → Change password.
2. Change the password to something secure. Its also recommended to change the username name.

### 📦 Step 6 – Configure Storage Node (Optional)

By default, FOG uses the local system for image storage:
 
 - Images are saved under: /images

To adjust or add more storage:

1. Go to FOG Configuration → Storage Management
2. Edit or add a storage node
3. Ensure NFS and permissions are correctly set

### ✅ FOG Server is now ready!

You can:
- Register a host
- Capture/deploy an image
- Set up snap-ins or scripts


## 2. 🖼️ Capturing and deploying OS images using FOG

This section will show you how to:

 - Register a client machine with FOG.
 - Capture a disk image from that client.
 - Deploy that image to other machines over the network.

⚙️ Prerequisites

Before we begin, make sure you have:

 ✅ A computer or VM with an OS installed (this will be the source machine)<br>
 ✅ It must be connected to the same network as the FOG server<br>
 ✅ PXE boot enabled in BIOS/UEFI<br>
 ✅ Optionally sysprepped (for Windows) or generalized


### 📝 Step 1 – Register the Host (Client Machine)

1. Boot the client via PXE (Network Boot).
2. It should load FOG’s iPXE menu.
3. Choose:
    **“Perform Full Host Registration and Inventory”**
4. Follow the prompts to:
 - Set a hostname
 - Optionally assign it to a group
 - Finish registration

Once done, the client will shut down or return to the menu.


### 🗂 Step 2 – Create a New Image Definition

1. On your FOG server, log in to the web UI:
http://`<your-server-ip>`/fog/management

2. Go to:
Image Management → Create New Image

3. Fill in the form:

| Field              | Value                                            |
| ------------------ | ------------------------------------------------ |
| **Image Name**     | Something descriptive, e.g., `Windows10Base`     |
| **Image Path**     | Leave as default (auto-generates from the name)  |
| **OS Type**        | Choose the correct OS (e.g., Windows 10)         |
| **Image Type**     | `Single Disk - Resizable` (most common)          |
| **Partition Type** | `MBR` or `GPT`, depending on the client disk     |


4. Click Add to save.

### 🔁 Step 3 – Assign the Image to the Host

1. Go to Host Management → List All Hosts
2. Find the registered client
3. Edit it
4. Under “Host Image”, choose the image you just created
5. Save changes

### 🧠 Step 4 – Schedule a Capture Task
1. Go to Task Management → List All Hosts
2. Click the “Upload Image” (arrow up) icon next to the Host
3. This schedules the task

> ✅ You’ll see the Host marked as “Task in Progress.”
{: .prompt-tip}

### 🚀 Step 5 – Boot the Client via PXE to Start Capture

1. Boot the client again via PXE
2. It will automatically start the upload process
3. FOG will use Partclone to image the disk and save it to the server’s /images/<image_name> folder

> This may take several minutes, depending on image size and network speed.
{: .prompt-warning}


### 🖥 Step 6 – Deploy the Image to Other Machines

To deploy the captured image:
1. Register the target client the same way as before
2. Assign it to the same image definition
3. Go to Task Management, select Deploy Image
4. Boot the target via PXE
5. Watch it deploy the whole image in minutes 🎉

### 🛠️ Extra Tips

 - For Windows, prepare with Sysprep before capturing
 - You can set up Groups to mass-deploy to multiple hosts
 - Enable Multicast in FOG if deploying to many machines at once
 - Use Snap-ins for post-deployment automation (apps, settings)


## 3. 🧰 Integrating netboot.xyz into the FOG PXE/iPXE menu

We’ll add a new boot menu option to the FOG PXE menu that chains to netboot.xyz. Adding this option gives your clients access to a wide range of OS installers and utilities directly over the internet.

#### 📍 Prerequisites:

 - Your PXE environment is working (as it is now).
 - Internet access is available from PXE-booted clients (they'll download netboot.xyz content directly).
 - undionly.kpxe or ipxe.efi must support HTTP (FOG-provided ones do).
 - FOG server running

### ⚙️ Step 1 - Choose the Right iPXE Loader

netboot.xyz provides specific iPXE binaries depending on architecture:

**Legacy BIOS:** `netboot.xyz-undionly.kpxe`

**EFI:** `netboot.xyz.efi`

Place them in your TFTP root located at /tftpboot (or FOG's TFTP directory):

```bash
cd /tftpboot
sudo wget https://boot.netboot.xyz/ipxe/netboot.xyz-undionly.kpxe
sudo wget https://boot.netboot.xyz/ipxe/netboot.xyz.efi
sudo chown fogproject:root netboot.xyz-undionly.kpxe netboot.xyz.efi
sudo chmod 655 netboot.xyz-undionly.kpxe netboot.xyz.efi
```

### 🧩 Step 2: Configure the Custom PXE Menu Entry in FOG
1. Log in to the FOG Web UI → Navigate to FOG Configuration → iPXE New Menu Entry.
2. Click Add Menu Item, and fill in:
```yaml
Menu Text: Boot netboot.xyz
Kernel: chain http://<fog-ip>/ipxe/netboot.xyz.efi
Initrd: (leave blank)
Arguments: (leave blank)
```
This config will let iPXE to load netboot.xyz from the upstream CDN.


3. Save and optionally reorder to appear where you'd like in the menu.


## 4. Testing in BIOS and UEFI environments

## 5. Optional enhancements (secure boot, custom menus, HTTPS)



