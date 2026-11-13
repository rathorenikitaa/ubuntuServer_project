# ubuntuServer_project

# Ubuntu Infrastructure: Secure Remote Access & User Management

# Project Overview
This documentation outlines the deployment of a secure remote administration environment. The project involves configuring an OpenSSH server on an Ubuntu host and establishing secure remote connections from a Windows client to perform foundational user management and system maintenance tasks.

Environment Architecture
 * Host Server: Ubuntu Linux
 * Client Workstation: Windows 10 / 11
 * Core Technologies: OpenSSH, Linux User Management Utilities


1. SSH Server Deployment (Ubuntu)
To enable headless remote administration, the OpenSSH server was installed and secured on the target Ubuntu machine.
Deployment Execution:
 * Package Installation: The standard openssh-server package was retrieved and installed via the APT package manager.

   Command: sudo apt update
   <img width="1469" height="539" alt="Screenshot 2026-11-12 172721" src="https://github.com/user-attachments/assets/e3467f83-4b71-471b-a02d-7ae1a019a8bc" />
 * Service Initialization: The SSH service was activated and configured to persist across system reboots to maintain infrastructure stability.
Command: sudo apt install openssh-server -y

 sudo systemctl enable ssh
<img width="1112" height="332" alt="image" src="https://github.com/user-attachments/assets/66d982c6-0fb1-4477-9265-c57f79f9e45a" />

 * Service Verification: The operational status of the daemon was validated to ensure it was actively listening for connections.
Command: systemctl status ssh
<img width="1059" height="533" alt="Screenshot 2026-11-12 173412" src="https://github.com/user-attachments/assets/e2c4ba9f-2ae1-4837-9831-326e0be45833" />

 * Network Security: The local firewall (UFW) was updated to explicitly permit inbound SSH traffic on port 22, ensuring legitimate remote traffic is not dropped.
   sudo ufw allow ssh


2. Remote Client Connection (Windows)
Remote access was established using the native Windows SSH client, simulating a standard engineering workflow for server maintenance.
Connection Protocol:
 * A terminal session was initiated from the Windows workstation targeting the Ubuntu server's IPv4 address.

command : ssh username@<ubuntu_ip_address>

 * Upon initial connection, the server's ECDSA key fingerprint was verified and appended to the Windows client's known_hosts file to establish cryptographic trust.
 * Password authentication was utilized to access the primary administrative account via the remote shell.
3. User Lifecycle Management
Following the successful remote connection, user administration procedures were executed directly from the Windows client to implement role-based access on the Linux host.
Execution Steps:
 * Account Provisioning: A secondary user account (newadmin) was provisioned. The utility automatically generated the required home directory structure and prompted for password credentials.
   sudo adduser newadmin

 * Privilege Escalation Assignment: The newly created account was appended to the sudo group, granting it administrative capabilities for future system configuration tasks.
   sudo usermod -aG sudo newadmin

 * Verification: The active session was transitioned to the newly created account to validate functionality and test elevated permissions.
   su - newadmin
sudo whoami

   (Expected output confirmed as root, validating successful sudo group assignment).
Security & Maintenance Principles Applied
 * Deployed robust remote management infrastructure, eliminating reliance on physical console access.
 * Enforced the principle of least privilege by provisioning standard user accounts for daily operations and restricting direct root access.
 * Ensured clean structural organization of services and firewalls to prioritize long-term system maintainability.
