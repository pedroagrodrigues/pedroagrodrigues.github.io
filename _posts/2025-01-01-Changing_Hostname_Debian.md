---
title: Changing Hostname
date: 2025-01-02 00:00:00
categories: [OS,System Administration]
tags: [Linux, Terminal, OS, Debian, Hostname, Networking]
---

> These commands work on  Debian and Ubuntu machines. It might work on some other distributions.
{: .prompt-tip}

### Step 1: Check the Current Hostname

Before changing the hostname, check the current hostname of your system using the following command:

```bash
hostnamectl
```
{: .nolineno }

or

```bash 
hostname
```
{: .nolineno }



### Step 2: Change the Hostname Temporarily (Until Reboot)
If you want to change the hostname temporarily (it will revert after a reboot), use the following command:

```bash
sudo hostname new-hostname
```
{: .nolineno }

Replace new-hostname with your desired hostname.

To verify the change:

```bash
hostname
```
{: .nolineno }

### Step 3: Change the Hostname Permanently
To change the hostname permanently, follow these steps:

Edit the `/etc/hostname` file:

```bash
sudo nano /etc/hostname
```
{: .nolineno }

Replace the existing hostname with your new hostname, then save and exit the file.


Open the `/etc/hosts` file, find the line that starts with `127.0.0.1` (or `127.0.1.1`) and replace the old hostname with the new one. 
For example:

```bash
sudo nano /etc/hosts
127.0.1.1 new-hostname
```
{: .nolineno }


Save and exit the file.

Apply the Changes

Use the `hostnamectl` command to apply the new hostname:

```bash
sudo hostnamectl set-hostname new-hostname
```
{: .nolineno }

### Step 4: Verify the Changes
To confirm that the hostname has changed, run the following:

```bash
hostnamectl
```
or

```bash 
hostname
```
{: .nolineno }

You should see the new hostname displayed.


### Step 5: Reboot the System (Optional)
To ensure the changes are fully applied, reboot your system:

```bash
sudo reboot
```
{: .nolineno }

After the reboot, verify the hostname again to confirm if the changes persisted.

> **Additional Notes:** Ensure the new hostname follows standard naming conventions (e.g., no spaces, special characters, or underscores). If your system is part of a network, ensure the new hostname does not conflict with other devices.
{: .prompt-info}

## Conclusion

Changing the hostname on a Debian system is a straightforward process that can be done temporarily or permanently, depending on your needs. Following the steps outlined in this tutorial, you can easily update your system’s hostname to reflect its purpose better, improve network identification, or meet organizational requirements.

Remember:

Always verify the changes using commands like `hostnamectl` or `hostname`.

Ensure the new hostname adheres to standard naming conventions to avoid potential issues.

If your system is part of a network, double-check that the new hostname is unique and does not conflict with other devices.

With these steps, you can confidently manage and customize your Debian system’s hostname. Whether working on a personal project or managing a server, this skill will help you keep your systems organized and easily identifiable. Happy computing! 🚀