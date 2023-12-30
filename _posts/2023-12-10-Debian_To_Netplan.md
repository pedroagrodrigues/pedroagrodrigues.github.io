---
title: Converting Debian 10/11/12 to Netplan.io
date: 2023-12-10 14:20:00
categories: [Networking, Linux]
tags: [linux, networking, static ip, network configuration, terminal, debian]
---

# Netplan Migration

The default network is typically systemd-networkd or ifupdown for most Debian-based systems, including Debian itself. The choice between these two can depend on the specific version and configuration of the system.
Ubuntu migrated to Netplan on its version 17.10. Since then, Netplan has gained more and more popularity

This tutorial will guide you through migrating a Debian-based system from the default ifupdown network manager to Netplan.

## What is a network manager?

A network manager is essential for simplifying network configuration and enhancing user experience on computer systems. It provides user-friendly interfaces for dynamic configuration, automates settings like IP addresses and DNS through DHCP, manages wireless connections, facilitates troubleshooting with logging, and integrates with system initialization processes. Network managers support VPNs, enhance security with firewall integration, ensure consistency across Linux distributions, and offer APIs for automation. They are crucial in streamlining network-related tasks and maintaining a standardized, user-friendly network environment.

### Netplan

**Netplan** is a _YAML-based_ network configuration tool in Linux, commonly used in distributions like Ubuntu. It simplifies network setup by providing a human-readable configuration format. Netplan abstracts backend configurators (e.g., systemd-networkd), maintains a consistent syntax, supports dynamic changes, and integrates with systemd for synchronization during system initialization. It offers backward compatibility with traditional tools, making it easier for users to transition. Configuration is done by editing a YAML file in `/etc/netplan/`, followed by applying changes with the netplan apply command.

### Ifupdown

**ifupdown** is a traditional network configuration tool in Linux. It relies on the `/etc/network/interfaces` file, written in a simple text format. Unlike Netplan, ifupdown does not use YAML. It provides a straightforward way to define network interfaces, IP addresses, and other parameters. Changes to configurations typically require restarting the networking service or rebooting the system. ifupdown is known for its simplicity and is often used in Debian-based systems, offering a familiar setup for users accustomed to traditional network configuration methods. To configure network interfaces using ifupdown, users edit the `/etc/network/interfaces` file and then use commands like `ifup` and `ifdown` to apply changes or bring interfaces `up/down`.

#### Why switch to netplan?

Netplan and ifupdown are Linux network configuration tools with distinct characteristics. Netplan uses YAML for configuration, offering a structured and dynamic setup, while ifupdown relies on a more straightforward text format. Netplan acts as a front-end to backends like systemd-networkd, supports dynamic changes and integrates well with systemd. Ifupdown is known for its specific syntax, requiring restarts for changes, and is commonly used in Debian-based systems. The choice depends on user preference, system requirements, and the Linux distribution. Netplan is prevalent in systemd-based systems like Ubuntu, while ifupdown is traditional, often found in Debian.

## Step 1 - Installing Netplan.io

> It is recommended to save your configuration on all steps. If you are using a virtual machine, you can take a snapshot. Proceed at your own risk.
{: .prompt-warning}

We start by installing netplan.io.

```bash
sudo apt update && sudo apt install netplan.io -y
```

## Step 2 - Enable Network Services

The network services we require are masked to unmask and enable them to run the following:

```bash
sudo systemctl unmask systemd-networkd.service;
sudo systemctl unmask systemd-resolved.service;
sudo systemctl enable systemd-networkd.service;
sudo systemctl mask networking;
sudo systemctl enable systemd-resolved.service;
```

> If you get an error about systemd-resolved not being found/installed. Install it before proceeding:
{: .prompt-danger}

```bash
sudo apt install systemd-resolved -y
```

## Step 3 - Migrate to Netplan.io

Now that we have installed netplan.io and enabled the required services, let's perform the migration.

```bash
sudo ENABLE_TEST_COMMANDS=1 netplan migrate && sudo netplan try
```

If everything goes smoothly, you should get a similar output:

```plaintext
migration complete, wrote /etc/netplan/10-ifupdown.yaml
renaming /etc/network/interfaces to /etc/network/interfaces.netplan-converted

Do you want to keep these settings?


Press ENTER before the timeout to accept the new configuration


Changes will revert in 110 seconds
```

At this point, you should press `enter` to save the changes.

Fix the permissions for the newly created netplan file:

```bash
sudo chmod 600 /etc/netplan/*
```

> If you are using a static IP, updating the configuration is a good practice.
{: .prompt-tip}

To achieve this you have to edit the file inside `/etc/netplan/`. Heres an example of a static configuration:

```yaml
network:
  version: 2
  ethernets:
    ens18:
      addresses:
        - <your ip>/<mask>
      routes:
        - to: default
          via: <your gateway ip>
      nameservers:
        addresses:
          - 1.1.1.1
          - 1.0.0.1
          - 8.8.8.8
          - 8.8.4.4
        search:
          - <local domain>
```

> Search is not mandatory. You may remove it if you don't have a domain.
{: .prompt-info}

## Step 4 - Reboot

We recommend that you reboot your system at this point. The system should boot using Netplan configurations.

## Step 5 - Uninstall Old Packages

Now that you have rebooted your machine and everything is ready remove the `ifupdown` and `resolvconf` packages:

```bash
sudo apt purge ifupdown resolvconf -y && sudo rm -rf /etc/network
```

Lastly, create a new link for the `resolv.conf`:

```bash
sudo ln -sf /run/systemd/resolve/stub-resolv.conf /etc/resolv.conf
```

### Testing

If everything works fine, you should be able to ping the internet:

```bash
ping google.com
```

## Conclusion

This tutorial explored migrating a Debian machine from the traditional ifupdown networking configuration to the modern and flexible netplan.io.

Netplan.io’s YAML-based configuration offers a more human-readable and extensible approach, enabling users to define complex network setups easily. While the migration may require users to adapt to a new syntax, the benefits in terms of clarity and support for advanced networking features make it a worthwhile endeavour.

Following the step-by-step guide, you’ve learned how to convert from ifupdown to a netplan configuration, setting up a static IP, and verify the network connectivity. Remember to back up your existing configuration before starting the migration, and take the time to review and understand your network requirements.

With the migration complete, your Debian machine is now equipped with a modern networking tool that aligns with current best practices, making it easier to manage and adapt to evolving network configurations. Explore additional netplan features and tailor your network setup to meet your needs.
