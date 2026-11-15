#   Ubuntu Server Lab: Troubleshooting Log
# Issue 1: openssh-server has no installation candidate
# Symptom:
   Error: Package 'openssh-server' has no installation candidate
       Error: Unable to locate package openssh-server-y
        openssh-server: command not found
Root cause (two parts):
1.  The package index was empty or outdated because the VM could not reach the Ubuntu repositories (see Issue 4).
2.  Typing mistakes: openssh-server-y (no space before -y) and running openssh-server -y without apt install in front of it.
 Correct command:
sudo apt update
sudo apt install openssh-server -y
sudo systemctl status ssh
 Lesson: No installation candidate usually means the package list is empty, which usually means no internet.

# Issue 2: apt update fails with Temporary failure resolving
 
 Symptom: Every repository line failed with Temporary failure resolving archive.ubuntu.com and security.ubuntu.com.

ip a showed only one real adapter (ens34, 192.168.73.10/24).
That is a host-only style address, so there was no path to the internet.

Root cause: A host-only network only connects the VM to the host computer and other VMs. It never has internet access.
Lesson: Used two adapters for a proper lab: NAT for internet (DHCP) and host-only for management (static IP).

# Issue 3: Ping to 192.168.73.1 returns Destination Host Unreachable

ping 192.168.73.10  ->  17 received, 0% loss      (this is the VM's own address)
ping 192.168.73.1   ->  Destination Host Unreachable, 100% loss
Reading the result:
• Pinging 192.168.73.10 only proves the VM can reach itself.
• Unreachable coming from my own address means the VM sent an ARP request and nobody answered. No device owns 192.168.73.1 on that network.

Lesson: Reading who is replying. A reply from  own address means the target did not answer.

# Issue 4: Pending kernel upgrade after installing SSH

Pending kernel upgrade!
Running kernel version: 7.0.0-30-generic
Expected kernel version: 7.0.0-43-generic

The package install also brought a newer kernel, but the machine keeps running the old one until restart. openssh-server was set up correctly (sshd user created, ssh.socket and keygen services enabled).

Fix: sudo rebooted when convenient. This was a notice, not an error.

# Issue 5: No internet on VMware because of a static gateway on the host-only adapter

Symptom: apt update failed again after moving to VMware, and ping 8.8.8.8 showed 100% packet loss.

Root cause: ens34 had a static IP and a static default gateway (192.168.73.1), but that host-only network has no router to the internet. The only default route led nowhere. There was also no DNS server configured, which produced the resolving errors.

<t> <br>Design rule that came out of this:
Adapter
Purpose
Addressing
Default route
NAT (ens33)
Internet
DHCP
Yes
Host-only (ens34)
Lab and SSH access
Static 192.168.73.10/24
No gateway <br> <t>

# Issue 6: ens33 stays DOWN
Symptom: After sudo ip link set ens33  was up and sudo netplan apply, ip -br a still showed ens33 as DOWN with no IP, and with only host-only connected the VM got no internet address at all.

Root cause: Ubuntu could switch the interface on, but VMware had not connected the virtual network cable (the adapter was not set to Connected). The operating system cannot fix a cable that the hypervisor leaves unplugged.
Fix:
1.  VM > Removable Devices > Network Adapter: make sure Connected is ticked.
2.  VM > Settings > Network Adapter: tick Connected and Connect at power on, and set the type to NAT.
3.  Checked that the VMware NAT Service and VMware DHCP Service are running on Windows (services.msc), and that VMnet8 exists in the Virtual Network Editor.

Result: ens33 came UP, received an address, and internet access worked, so apt update and the SSH install could proceed.
Lesson: Troubleshoot in layers. Layer 1 (is the cable or adapter connected?) comes before addressing, routing and DNS.
Final working configuration

/etc/netplan/50-cloud-init.yaml (spaces only, never tabs):
network:
  version: 2
  ethernets:
    ens33:
      dhcp4: true
    ens34:
      addresses:
        - 192.168.73.10/24
Apply and verify:
sudo netplan apply
ip -br a
ip route
ping -c 3 8.8.8.8
ping -c 3 google.com
sudo apt update
sudo systemctl status ssh

Connect from Windows PowerShell:
ssh nikita@192.168.73.10


# Troubleshooting method I now follow
1.  Is the link up? ip -br a and ip link show (look for DOWN or NO-CARRIER).
2.  Do I have an IP? ip a.
3.  Do I have a default route? ip route (look for default via).
4.  Can I reach an IP outside? ping -c 3 8.8.8.8.
5.  Does DNS work? ping -c 3 google.com.
6.  Does the service work? systemctl status, then apt update.
