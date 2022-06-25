# ansible_cumulus
![](RackMultipart20220411-4-12wtqc_html_23768ca39a132272.png) ![](RackMultipart20220411-4-12wtqc_html_e2db356408442ac6.png) ![Shape1](RackMultipart20220411-4-12wtqc_html_ee2b371845eab381.gif)



# ­

# Table of Content :

[1Get in switching 3](#_Toc99738012)

[1.1Overview 3](#_Toc99738013)

[1.2Bonding with 802.3ad aka LACP 3](#_Toc99738014)

[1.3Vlans 20, 30, 90 3](#_Toc99738015)

[1.4VLSM Addressing 4](#_Toc99738016)

[1.5ConfigMgmt with Ansible and NCLU 4](#_Toc99738017)

[1.6Hosts Configuration 4](#_Toc99738018)

[1.6.1NFS 4](#_Toc99738019)

[1.6.2Mounting 5](#_Toc99738020)

[1.7Post-config verification 5](#_Toc99738021)

[1.7.1Bond 5](#_Toc99738022)

[1.7.2Trunk mode &amp; Access mode &amp; SVI 5](#_Toc99738023)

[1.7.3Port Security &amp; macof 6](#_Toc99738024)

[1.7.4ACL 6](#_Toc99738025)

[1.8Failover testing 6](#_Toc99738026)

[2Access Layer Redundancy 7](#_Toc99738027)

[2.1Overview 7](#_Toc99738028)

[2.2MLAG 7](#_Toc99738029)

[2.2.1Config verification 7](#_Toc99738030)

[2.2.2Failover testing 9](#_Toc99738031)

[2.3Spaning-tree 12](#_Toc99738032)

[2.3.1Portfast (namedPortAdminEdge )&amp; BPDUGuard 12](#_Toc99738033)

[2.3.2States/roles Analysis 12](#_Toc99738034)

[3Router Redundancy 15](#_Toc99738035)

# 1­­­Get in switching

## 1.1Overview

![](RackMultipart20220411-4-12wtqc_html_38616dbed19c0f01.png)

## 1.2Bonding with 802.3ad aka LACP

_Proposal Bonding improves the performance and add links redundancy as well. In multiple scenarios can possibly make maintenance smooth._

_Scenario: distribution switch uses pluggable SFP modules, first one in Bond0 went down, due to the bonding the access switch remains reachable. In addition, the maintenance can take place easily, users won&#39;t be impacted._

## 1.3Vlans 20, 30, 90

Vlan 90 will separate server from other users vlans which are 20, 30.

Vlans help basically to reduce broadcast domain in one switch, it operates on Layer 2 and when the switch receives a frame in an access port, a vlan tag is added, then the frame is forwarded only in access ports configured with same vlan or in trunk ports which accepts this vlan. As a result, the communication is isolated. Additional advantage is vlans represent services or whatever we would like to, which gives the ability to block or permit communication between services based on network IP, using ACL or Firewall.

**Access mode:** used for endpoints, accept one Vlan, the port in this mode will label frames received with his vlan id.

**Trunk mode:** Should be set in ports connected to other switches. Let go through all vlans accepted in the configuration of the port.

**SVI:** virtual interface, an IP can be assigned to it. Used a gateway for vlan to get connected with other vlan or the rest of the network including Internet.

## 1.4VLSM Addressing

**Network**  **IP**** :** 192.168.0.0/22 supports 4 subnets with 254 assignable IP each.

| **Subnet Name ** | **Address ** | **Mask ** | **Assignable Range ** |
| --- | --- | --- | --- |
| **Server – VLAN 90 ** | 192.168.0.0  | /24  | 192.168.0.1 - 192.168.0.254  |
| **MGMT - VLAN 99 ** | 192.168.1.0  | /24  | 192.168.1.1 - 192.168.1.254  |
| **VLAN 20 ** | 192.168.2.0  | /24  | 192.168.2.1 - 192.168.2.254  |
| **Vlan 30 ** | 192.168.3.0  | /24  | 192.168.3.1 - 192.168.3.254  |

## 1.5ConfigMgmt with Ansible and NCLU

I&#39;ve started by preparing an inventory and ansible config file, then installed the network module from ansible galaxy, I need to get NCLU from it which will permit the configuration of ansible.

**$ ansible-galaxy collection install community.network**

 ![Shape2](RackMultipart20220411-4-12wtqc_html_38abfffc5284bec3.gif)

**Please find attached my playbooks.**

**You can get from Github as well: @alakhel/ansible\_cumulus**

[alakhel/ansible\_cumulus (github.com)](https://github.com/alakhel/ansible_cumulus)

## 1.6Hosts Configuration

### 1.6.1NFS

**# yum install nfs-utils\***

**# systemctl enable –now rpcbind**

**# echo &quot;**

**/netshare/host1 192.168.2.2(rw,sync,no\_root\_squash)**

**/netshare/host2 192.168.3.2(rw,sync,no\_root\_squash)**

&quot; **\&gt;\&gt;/etc/exports**

 ![Shape3](RackMultipart20220411-4-12wtqc_html_53e7ceb820df4095.gif)

### 1.6.2Mounting

![](RackMultipart20220411-4-12wtqc_html_ee9abee63e61a5aa.png)

## 1.7Post-config verification

### 1.7.1­­­­Bond

![](RackMultipart20220411-4-12wtqc_html_6218041e9338d1fb.png)

### 1.7.2Trunk mode &amp; Access mode &amp; SVI

![](RackMultipart20220411-4-12wtqc_html_a2baaf692cf6de9d.png)

![](RackMultipart20220411-4-12wtqc_html_9ff9022749cee474.png)

![](RackMultipart20220411-4-12wtqc_html_7453911027584ed5.png)

### 1.7.3Port Security &amp; macof

Helps mitigate CAM flood attacks which can take the switch down (Netd)

It&#39;s easy to take down a switch which is vulnerable. In real situation any random person can infiltrate somehow inside the company (or anyone with legitimate access), prepare a Raspi then connect to a any RG45 socket, taking a switch or a stack of switches can make an entire network down. What if the security cameras are connected to the same Switch? …

I connected my Kali box to the SWA2:

**macof** send frames with ram mac address each time the switch will try to change its CAM (cisco term) table which take the service down.

![](RackMultipart20220411-4-12wtqc_html_63549785f9a315e6.png)

After few seconds:

![](RackMultipart20220411-4-12wtqc_html_be942ba5c1918906.png)

![](RackMultipart20220411-4-12wtqc_html_fd26a068fcc79e2b.png)

### 1.7.4ACL

## 1.8Failover testing

# Access Layer Redundancy

## 1.9 ![](RackMultipart20220411-4-12wtqc_html_c1f45eb0bcd3fd70.png)Overview

## 1.10MLAG

Use bond between a server and access switch still great solution to improve performance and have a redundant port, However the access switch is a single point of failure. MLAG resolve this issue by connecting a single server to two access switches.

Both switches will appear as one for the host, using LACP the host will have a bond which it&#39;s physical link connected two switches. MLAG will manage these links from switch side.

### 1.10.1Config verification

#### 1.10.1.1State

Logs for the clag dual link setup:

![](RackMultipart20220411-4-12wtqc_html_612daf14c1c7825a.png)

![](RackMultipart20220411-4-12wtqc_html_93602de69898d760.png)

Clag state, SWA1 is primary, SWA2 is secondary:

![](RackMultipart20220411-4-12wtqc_html_7e0811f306cb7e97.png)

![](RackMultipart20220411-4-12wtqc_html_d5825bcf294f1e0f.png)

#### 1.10.1.2ping

Ping from host1 192.168.10.1 went through SWA1:

![](RackMultipart20220411-4-12wtqc_html_7e718c751fe3a4c4.png)

On SWD1:

![](RackMultipart20220411-4-12wtqc_html_ab260e07900fb4fb.png)

### 1.10.2Failover testing

#### 1.10.2.1Case 1: take the switch down

I launched continuous ping to measure packet loss during the failover.

**State 0:** ping going through SWA1. Just like the previous part above.

**State 1:** take down SWA1 and watch ping on Host1

![](RackMultipart20220411-4-12wtqc_html_996d4302bfdb2742.png)

So, ping stop working, I check in SWA2 if there&#39;s anything, I found that it works correctly However Host1 is not accepting frames.

After some hours of troubleshooting, I activated promisc mode and problem solved. I&#39;m not sure what is the root cause, all theory causes I&#39;ve elaborated and checked was OK. I have no firewall activated; the bridge macs tables have the correct MAC addresses.

![](RackMultipart20220411-4-12wtqc_html_6b83dd6cf8fd0251.png)

Below is the same MAC for bond0 in host1:

![](RackMultipart20220411-4-12wtqc_html_21aad2d9800d67be.png)

Below is the capture of swp3 which connected to host 1, as the alias table show:

![](RackMultipart20220411-4-12wtqc_html_d7dcd220e21dc44.png)

![](RackMultipart20220411-4-12wtqc_html_66ecfbfdfb69e042.png)

That&#39;s enough to assume that the problem is caused by VMware virtualisation, I found a similar article on VMware community site, but it was about centos on ESXI.

**State 2:** take back SWA1, enable promisc mode, then run the failover again

Ping resumed after powering up back the SWA1.

Activate promisc mode:

![](RackMultipart20220411-4-12wtqc_html_ed8e80b790f12341.png)

I&#39;ve restarted ping to have a correct packet loss rate during failover.

![](RackMultipart20220411-4-12wtqc_html_aefe2d0758829d34.png)

So, the traffic switched to switch2 with 17% packet loss.

#### 1.10.2.2Wireshark captures analysis

Before I start, I should mention that the time I did this failover test and capture it, I still didn&#39;t change any STP configuration.

## 1.11Spaning-tree

Adding a peerlink between both switches will cause a loop and STP by default helps resolve it. But the problem is that we lost one bonding link which could be used to improve performance.

So, in part 2 I&#39;ll just try to **reduce convergence time**. However, in the next one, I&#39;ll try to avoid it.

### 1.11.1Portfast (namedPortAdminEdge )&amp; BPDUGuard

Portfast will make the port go directly to Forwarding, BPDU Guard will reject BPDU. It&#39;s a must.

Below is an example of these two options activated

![](RackMultipart20220411-4-12wtqc_html_71e1418fb218b808.png)

### 1.11.2States/roles Analysis

![](RackMultipart20220411-4-12wtqc_html_6f0949a857fcab4d.png)

![](RackMultipart20220411-4-12wtqc_html_71d175a66c7ec6fa.png)

![](RackMultipart20220411-4-12wtqc_html_b7fc2cac4120a417.png)

The SWD is root bridge, SWA1 have its bond0(uplink) as a root, SWA2 have the peerlink as root.

Therefore, the bond0 of SWA2 is blocked to avoid the loop.

![](RackMultipart20220411-4-12wtqc_html_c1f45eb0bcd3fd70.png)

# Router Redundancy
