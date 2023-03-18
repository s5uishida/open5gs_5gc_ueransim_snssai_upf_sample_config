# Open5GS 5GC & UERANSIM UE / RAN Sample Configuration - Select UPF based on S-NSSAI
This describes a very simple configuration that uses Open5GS and UERANSIM to select the UPF based on S-NSSAI.

---

<h2 id="conf_list">List of Sample Configurations</h2>

1. [One SGW-C/PGW-C, Multiple SGW-Us/PGW-Us and APNs](https://github.com/s5uishida/open5gs_epc_oai_sample_config)
2. [One SMF, Multiple UPFs and DNNs](https://github.com/s5uishida/open5gs_5gc_ueransim_sample_config)
3. [Select nearby UPF according to the connected gNodeB](https://github.com/s5uishida/open5gs_5gc_ueransim_nearby_upf_sample_config)
4. Select UPF based on S-NSSAI (this article)
5. [SCP Indirect communication Model C](https://github.com/s5uishida/open5gs_5gc_ueransim_scp_model_c_sample_config)
6. [VoLTE and SMS Configuration for docker_open5gs](https://github.com/s5uishida/docker_open5gs_volte_sms_config)
7. [Monitoring Metrics with Prometheus](https://github.com/s5uishida/open5gs_5gc_ueransim_metrics_sample_config)
8. [Framed Routing](https://github.com/s5uishida/open5gs_5gc_ueransim_framed_routing_sample_config)
---

<h2 id="misc">Miscellaneous Notes</h2>

- [Install MongoDB 6.0 and Open5GS WebUI](https://github.com/s5uishida/open5gs_install_mongodb6_webui)
---

<h2 id="toc">Table of Contents</h2>

- [Overview of Open5GS 5GC Simulation Mobile Network](#overview)
- [Changes in configuration files of Open5GS 5GC and UERANSIM UE / RAN](#changes)
  - [Changes in configuration files of Open5GS 5GC C-Plane](#changes_cp)
  - [Changes in configuration files of Open5GS 5GC U-Plane1](#changes_up1)
  - [Changes in configuration files of Open5GS 5GC U-Plane2](#changes_up2)
  - [Changes in configuration files of UERANSIM UE / RAN](#changes_ueransim)
    - [Changes in configuration files of RAN (gNodeB)](#changes_ran)
    - [Changes in configuration files of UE set to SST:1 and SD:0x000001 (IMSI-001010000000000)](#changes_ue_sd1)
    - [Changes in configuration files of UE set to SST:1 and SD:0x000002 (IMSI-001010000000000)](#changes_ue_sd2)
- [Network settings of Open5GS 5GC and UERANSIM UE / RAN](#network_settings)
  - [Network settings of Open5GS 5GC C-Plane](#network_settings_cp)
  - [Network settings of Open5GS 5GC U-Plane1](#network_settings_up1)
  - [Network settings of Open5GS 5GC U-Plane2](#network_settings_up2)
- [Build Open5GS and UERANSIM](#build)
- [Run Open5GS 5GC and UERANSIM UE / RAN](#run)
  - [Run Open5GS 5GC C-Plane](#run_cp)
  - [Run Open5GS 5GC U-Plane1 & U-Plane2](#run_up)
  - [Run UERANSIM (gNodeB)](#run_ran)
  - [Run UERANSIM (UE[SST:1, SD:0x000001] connects to U-Plane1)](#run_sd1)
  - [Run UERANSIM (UE[SST:1, SD:0x000002] connects to U-Plane2)](#run_sd2)
  - [Ping google.com going through DN=10.45.0.0/16 on U-Plane1 via uesimtun0](#ping_sd1)
  - [Ping google.com going through DN=10.46.0.0/16 on U-Plane2 via uesimtun1](#ping_sd2)
- [Changelog (summary)](#changelog)

---
<h2 id="overview">Overview of Open5GS 5GC Simulation Mobile Network</h2>

The following minimum configuration was set as a condition.
- The UE selects a pair of SMF and UPF based on S-NSSAI.

The built simulation environment is as follows.

<img src="./images/network-overview.png" title="./images/network-overview.png" width=1000px></img>

The 5GC / UE / RAN used are as follows.
- 5GC - Open5GS v2.6.1 (2023.03.17) - https://github.com/open5gs/open5gs
- UE / RAN - UERANSIM v3.2.6 (2023.03.17) - https://github.com/aligungr/UERANSIM

Each VMs are as follows.  
| VM # | SW & Role | IP address | OS | Memory (Min) | HDD (Min) |
| --- | --- | --- | --- | --- | --- |
| VM1 | Open5GS 5GC C-Plane | 192.168.0.111/24 <br> 192.168.0.112/24 <br> 192.168.0.113/24 | Ubuntu 22.04 | 1GB | 20GB |
| VM2 | Open5GS 5GC U-Plane1  | 192.168.0.114/24 | Ubuntu 22.04 | 1GB | 20GB |
| VM3 | Open5GS 5GC U-Plane2  | 192.168.0.115/24 | Ubuntu 22.04 | 1GB | 20GB |
| VM4 | UERANSIM RAN (gNodeB) | 192.168.0.131/24 | Ubuntu 22.04 | 1GB | 10GB |
| VM5 | UERANSIM UE | 192.168.0.132/24 | Ubuntu 22.04 | 1GB | 10GB |

AMF & SMF addresses are as follows.  
| NF | IP address | IP address on SBI | Supported S-NSSAI |
| --- | --- | --- | --- |
| AMF | 192.168.0.111 | 127.0.0.5 | SST:1, SD:0x000001 <br> SST:1, SD:0x000002 |
| SMF1 | 192.168.0.112 | 127.0.0.4 | SST:1, SD:0x000001 |
| SMF2 | 192.168.0.113 | 127.0.0.24 | SST:1, SD:0x000002 |

gNodeB Information (other information is default) is as follows.  
| IP address | Supported S-NSSAI |
| --- | --- |
| 192.168.0.131 | SST:1, SD:0x000001 <br> SST:1, SD:0x000002 |

Subscriber Information (other information is default) is as follows.  
**Note. Please select OP or OPc according to the setting of UERANSIM UE configuration files.**
| UE | IMSI | DNN | OP/OPc | S-NSSAI |
| --- | --- | --- | --- | --- |
| UE | 001010000000000 | internet | OPc | SST:1, SD:0x000001 <br> SST:1, SD:0x000002|

I registered these information with the Open5GS WebUI.
In addition, [3GPP TS 35.208](https://www.3gpp.org/DynaReport/35208.htm) "4.3 Test Sets" is published by 3GPP as test data for the 3GPP authentication and key generation functions (MILENAGE).

Each DNs are as follows.
| DN | S-NSSAI |  TUNnel interface of DN | DNN | TUNnel interface of UE | U-Plane # |
| --- | --- | --- | --- | --- | --- |
| 10.45.0.0/16 | SST:1 <br> SD:0x000001 | ogstun | internet | uesimtun0 | U-Plane1 |
| 10.46.0.0/16 | SST:1 <br> SD:0x000002 | ogstun | internet | uesimtun1 | U-Plane2 |

<h2 id="changes">Changes in configuration files of Open5GS 5GC and UERANSIM UE / RAN</h2>

Please refer to the following for building Open5GS and UERANSIM respectively.
- Open5GS v2.6.1 (2023.03.17) - https://open5gs.org/open5gs/docs/guide/02-building-open5gs-from-sources/
- UERANSIM v3.2.6 (2023.03.17) - https://github.com/aligungr/UERANSIM/wiki/Installation

<h3 id="changes_cp">Changes in configuration files of Open5GS 5GC C-Plane</h3>

- `open5gs/install/etc/open5gs/amf.yaml`
```diff
--- amf.yaml.orig       2023-02-18 16:56:58.000000000 +0900
+++ amf.yaml    2023-02-18 17:53:50.000000000 +0900
@@ -416,28 +416,31 @@
       - addr: 127.0.0.5
         port: 7777
     ngap:
-      - addr: 127.0.0.5
+      - addr: 192.168.0.111
     metrics:
       - addr: 127.0.0.5
         port: 9090
     guami:
       - plmn_id:
-          mcc: 999
-          mnc: 70
+          mcc: 001
+          mnc: 01
         amf_id:
           region: 2
           set: 1
     tai:
       - plmn_id:
-          mcc: 999
-          mnc: 70
+          mcc: 001
+          mnc: 01
         tac: 1
     plmn_support:
       - plmn_id:
-          mcc: 999
-          mnc: 70
+          mcc: 001
+          mnc: 01
         s_nssai:
           - sst: 1
+            sd: 000001
+          - sst: 1
+            sd: 000002
     security:
         integrity_order : [ NIA2, NIA1, NIA0 ]
         ciphering_order : [ NEA0, NEA1, NEA2 ]
```
- `open5gs/install/etc/open5gs/smf1.yaml`
```diff
--- smf.yaml.orig       2023-03-10 21:49:28.000000000 +0900
+++ smf1.yaml   2023-03-11 17:42:14.000000000 +0900
@@ -20,7 +20,7 @@
 #    domain: core,sbi,ausf,event,tlv,mem,sock
 #
 logger:
-    file: /root/open5gs/install/var/log/open5gs/smf.log
+    file: /root/open5gs/install/var/log/open5gs/smf1.log
 
 #
 #  o TLS enable/disable
@@ -602,20 +602,17 @@
       - addr: 127.0.0.4
         port: 7777
     pfcp:
-      - addr: 127.0.0.4
-      - addr: ::1
+      - addr: 192.168.0.112
     gtpc:
       - addr: 127.0.0.4
-      - addr: ::1
     gtpu:
-      - addr: 127.0.0.4
-      - addr: ::1
+      - addr: 192.168.0.112
     metrics:
       - addr: 127.0.0.4
         port: 9090
     subnet:
       - addr: 10.45.0.1/16
-      - addr: 2001:db8:cafe::1/48
+        dnn: internet
     dns:
       - 8.8.8.8
       - 8.8.4.4
@@ -624,7 +621,18 @@
     mtu: 1400
     ctf:
       enabled: auto
-    freeDiameter: /root/open5gs/install/etc/freeDiameter/smf.conf
+    freeDiameter: /root/open5gs/install/etc/freeDiameter/smf1.conf
+    info:
+      - s_nssai:
+          - sst: 1
+            sd: 000001
+            dnn:
+              - internet
+        tai:
+          - plmn_id:
+              mcc: 001
+              mnc: 01
+            tac: 1
 
 #
 #  <SBI Client>>
@@ -813,7 +821,8 @@
 #
 upf:
     pfcp:
-      - addr: 127.0.0.7
+      - addr: 192.168.0.114
+        dnn: internet
 
 #
 #  o Disable use of IPv4 addresses (only IPv6)
```
- `open5gs/install/etc/open5gs/smf2.yaml`
```diff
--- smf.yaml.orig       2023-03-10 21:49:28.000000000 +0900
+++ smf2.yaml   2023-03-17 21:10:44.184592280 +0900
@@ -20,7 +20,7 @@
 #    domain: core,sbi,ausf,event,tlv,mem,sock
 #
 logger:
-    file: /root/open5gs/install/var/log/open5gs/smf.log
+    file: /root/open5gs/install/var/log/open5gs/smf2.log
 
 #
 #  o TLS enable/disable
@@ -599,23 +599,20 @@
 #
 smf:
     sbi:
-      - addr: 127.0.0.4
+      - addr: 127.0.0.24
         port: 7777
     pfcp:
-      - addr: 127.0.0.4
-      - addr: ::1
+      - addr: 192.168.0.113
     gtpc:
-      - addr: 127.0.0.4
-      - addr: ::1
+      - addr: 127.0.0.24
     gtpu:
-      - addr: 127.0.0.4
-      - addr: ::1
+      - addr: 192.168.0.113
     metrics:
-      - addr: 127.0.0.4
+      - addr: 127.0.0.24
         port: 9090
     subnet:
-      - addr: 10.45.0.1/16
-      - addr: 2001:db8:cafe::1/48
+      - addr: 10.46.0.1/16
+        dnn: internet
     dns:
       - 8.8.8.8
       - 8.8.4.4
@@ -624,7 +621,18 @@
     mtu: 1400
     ctf:
       enabled: auto
-    freeDiameter: /root/open5gs/install/etc/freeDiameter/smf.conf
+    freeDiameter: /root/open5gs/install/etc/freeDiameter/smf2.conf
+    info:
+      - s_nssai:
+          - sst: 1
+            sd: 000002
+            dnn:
+              - internet
+        tai:
+          - plmn_id:
+              mcc: 001
+              mnc: 01
+            tac: 1
 
 #
 #  <SBI Client>>
@@ -813,7 +821,8 @@
 #
 upf:
     pfcp:
-      - addr: 127.0.0.7
+      - addr: 192.168.0.115
+        dnn: internet
 
 #
 #  o Disable use of IPv4 addresses (only IPv6)
```
- `open5gs/install/etc/open5gs/nssf.yaml`
```diff
--- nssf.yaml.orig      2023-02-18 16:56:58.000000000 +0900
+++ nssf.yaml   2023-02-18 17:54:48.000000000 +0900
@@ -313,6 +313,12 @@
         port: 7777
         s_nssai:
           sst: 1
+          sd: 000001
+      - addr: 127.0.0.10
+        port: 7777
+        s_nssai:
+          sst: 1
+          sd: 000002
 
 #
 #  <SBI Client>>
```
- `open5gs/install/etc/freeDiameter/smf1.conf`  
`smf1.conf` is equal to the original `smf.conf`.

- `open5gs/install/etc/freeDiameter/smf2.conf`
```diff
--- smf.conf.orig       2023-01-12 20:33:20.131297687 +0900
+++ smf2.conf   2023-01-12 22:22:40.352706816 +0900
@@ -79,7 +79,7 @@
 #ListenOn = "202.249.37.5";
 #ListenOn = "2001:200:903:2::202:1";
 #ListenOn = "fe80::21c:5ff:fe98:7d62%eth0";
-ListenOn = "127.0.0.4";
+ListenOn = "127.0.0.24";
 
 
 ##############################################################
```

<h3 id="changes_up1">Changes in configuration files of Open5GS 5GC U-Plane1</h3>

- `open5gs/install/etc/open5gs/upf.yaml`
```diff
--- upf.yaml.orig       2023-03-02 22:07:32.000000000 +0900
+++ upf.yaml    2023-03-11 17:45:02.000000000 +0900
@@ -196,12 +196,13 @@
 #
 upf:
     pfcp:
-      - addr: 127.0.0.7
+      - addr: 192.168.0.114
     gtpu:
-      - addr: 127.0.0.7
+      - addr: 192.168.0.114
     subnet:
       - addr: 10.45.0.1/16
-      - addr: 2001:db8:cafe::1/48
+        dnn: internet
+        dev: ogstun
     metrics:
       - addr: 127.0.0.7
         port: 9090
```

<h3 id="changes_up2">Changes in configuration files of Open5GS 5GC U-Plane2</h3>

- `open5gs/install/etc/open5gs/upf.yaml`
```diff
--- upf.yaml.orig       2023-03-02 22:18:20.000000000 +0900
+++ upf.yaml    2023-03-11 17:46:06.000000000 +0900
@@ -196,12 +196,13 @@
 #
 upf:
     pfcp:
-      - addr: 127.0.0.7
+      - addr: 192.168.0.115
     gtpu:
-      - addr: 127.0.0.7
+      - addr: 192.168.0.115
     subnet:
-      - addr: 10.45.0.1/16
-      - addr: 2001:db8:cafe::1/48
+      - addr: 10.46.0.1/16
+        dnn: internet
+        dev: ogstun
     metrics:
       - addr: 127.0.0.7
         port: 9090
```

<h3 id="changes_ueransim">Changes in configuration files of UERANSIM UE / RAN</h3>

<h4 id="changes_ran">Changes in configuration files of RAN (gNodeB)</h4>

- `UERANSIM/config/open5gs-gnb.yaml`
```diff
--- open5gs-gnb.yaml.orig       2022-07-03 13:06:43.000000000 +0900
+++ open5gs-gnb.yaml    2023-01-12 23:39:36.000000000 +0900
@@ -1,22 +1,25 @@
-mcc: '999'          # Mobile Country Code value
-mnc: '70'           # Mobile Network Code value (2 or 3 digits)
+mcc: '001'          # Mobile Country Code value
+mnc: '01'           # Mobile Network Code value (2 or 3 digits)
 
 nci: '0x000000010'  # NR Cell Identity (36-bit)
 idLength: 32        # NR gNB ID length in bits [22...32]
 tac: 1              # Tracking Area Code
 
-linkIp: 127.0.0.1   # gNB's local IP address for Radio Link Simulation (Usually same with local IP)
-ngapIp: 127.0.0.1   # gNB's local IP address for N2 Interface (Usually same with local IP)
-gtpIp: 127.0.0.1    # gNB's local IP address for N3 Interface (Usually same with local IP)
+linkIp: 192.168.0.131   # gNB's local IP address for Radio Link Simulation (Usually same with local IP)
+ngapIp: 192.168.0.131   # gNB's local IP address for N2 Interface (Usually same with local IP)
+gtpIp: 192.168.0.131    # gNB's local IP address for N3 Interface (Usually same with local IP)
 
 # List of AMF address information
 amfConfigs:
-  - address: 127.0.0.5
+  - address: 192.168.0.111
     port: 38412
 
 # List of supported S-NSSAIs by this gNB
 slices:
   - sst: 1
+    sd: 0x000001
+  - sst: 1
+    sd: 0x000002
 
 # Indicates whether or not SCTP stream number errors should be ignored.
 ignoreStreamIds: true
```

<h4 id="changes_ue_sd1">Changes in configuration files of UE set to SST:1 and SD:0x000001 (IMSI-001010000000000)</h4>

- `UERANSIM/config/open5gs-ue-sd1.yaml`
```diff
--- open5gs-ue.yaml.orig        2023-03-17 19:17:13.000000000 +0900
+++ open5gs-ue-sd1.yaml 2023-03-17 21:38:52.705311100 +0900
@@ -1,11 +1,11 @@
 # IMSI number of the UE. IMSI = [MCC|MNC|MSISDN] (In total 15 digits)
-supi: 'imsi-999700000000001'
+supi: 'imsi-001010000000000'
 # Mobile Country Code value of HPLMN
-mcc: '999'
+mcc: '001'
 # Mobile Network Code value of HPLMN (2 or 3 digits)
-mnc: '70'
+mnc: '01'
 # Routing Indicator
-routingIndicator: '0000'
+routingIndicator: '0001'
 
 # Permanent subscription key
 key: '465B5CE8B199B49FAA5F0A2EE238A6BC'
@@ -22,7 +22,7 @@
 
 # List of gNB IP addresses for Radio Link Simulation
 gnbSearchList:
-  - 127.0.0.1
+  - 192.168.0.131
 
 # UAC Access Identities Configuration
 uacAic:
@@ -44,15 +44,17 @@
     apn: 'internet'
     slice:
       sst: 1
+      sd: 0x000001
 
 # Configured NSSAI for this UE by HPLMN
 configured-nssai:
   - sst: 1
+    sd: 0x000001
 
 # Default Configured NSSAI for this UE
 default-nssai:
   - sst: 1
-    sd: 1
+    sd: 0x000001
 
 # Supported integrity algorithms by this UE
 integrity:
```

<h4 id="changes_ue_sd2">Changes in configuration files of UE set to SST:1 and SD:0x000002 (IMSI-001010000000000)</h4>

- `UERANSIM/config/open5gs-ue-sd2.yaml`
```diff
--- open5gs-ue.yaml.orig        2023-03-17 19:17:13.000000000 +0900
+++ open5gs-ue-sd2.yaml 2023-03-17 21:38:59.233347117 +0900
@@ -1,11 +1,11 @@
 # IMSI number of the UE. IMSI = [MCC|MNC|MSISDN] (In total 15 digits)
-supi: 'imsi-999700000000001'
+supi: 'imsi-001010000000000'
 # Mobile Country Code value of HPLMN
-mcc: '999'
+mcc: '001'
 # Mobile Network Code value of HPLMN (2 or 3 digits)
-mnc: '70'
+mnc: '01'
 # Routing Indicator
-routingIndicator: '0000'
+routingIndicator: '0002'
 
 # Permanent subscription key
 key: '465B5CE8B199B49FAA5F0A2EE238A6BC'
@@ -22,7 +22,7 @@
 
 # List of gNB IP addresses for Radio Link Simulation
 gnbSearchList:
-  - 127.0.0.1
+  - 192.168.0.131
 
 # UAC Access Identities Configuration
 uacAic:
@@ -44,15 +44,17 @@
     apn: 'internet'
     slice:
       sst: 1
+      sd: 0x000002
 
 # Configured NSSAI for this UE by HPLMN
 configured-nssai:
   - sst: 1
+    sd: 0x000002
 
 # Default Configured NSSAI for this UE
 default-nssai:
   - sst: 1
-    sd: 1
+    sd: 0x000002
 
 # Supported integrity algorithms by this UE
 integrity:
```

<h2 id="network_settings">Network settings of Open5GS 5GC and UERANSIM UE / RAN</h2>

<h3 id="network_settings_cp">Network settings of Open5GS 5GC C-Plane</h3>

Add IP addresses for SMF1 and SMF2.
```
ip addr add 192.168.0.112/24 dev enp0s8
ip addr add 192.168.0.113/24 dev enp0s8
```
**Note. `enp0s8` is the network interface of `192.168.0.0/24` in my VirtualBox environment.
Please change it according to your environment.**

<h3 id="network_settings_up1">Network settings of Open5GS 5GC U-Plane1</h3>

First, uncomment the next line in the `/etc/sysctl.conf` file and reflect it in the OS.
```
net.ipv4.ip_forward=1
```
```
# sysctl -p
```
Next, configure the TUNnel interface and NAPT.
```
ip tuntap add name ogstun mode tun
ip addr add 10.45.0.1/16 dev ogstun
ip link set ogstun up

iptables -t nat -A POSTROUTING -s 10.45.0.0/16 ! -o ogstun -j MASQUERADE
```

<h3 id="network_settings_up2">Network settings of Open5GS 5GC U-Plane2</h3>

First, uncomment the next line in the `/etc/sysctl.conf` file and reflect it in the OS.
```
net.ipv4.ip_forward=1
```
```
# sysctl -p
```
Next, configure the TUNnel interface and NAPT.
```
ip tuntap add name ogstun mode tun
ip addr add 10.46.0.1/16 dev ogstun
ip link set ogstun up

iptables -t nat -A POSTROUTING -s 10.46.0.0/16 ! -o ogstun -j MASQUERADE
```

<h2 id="build">Build Open5GS and UERANSIM</h2>

Please refer to the following for building Open5GS and UERANSIM respectively.
- Open5GS v2.6.1 (2023.03.17) - https://open5gs.org/open5gs/docs/guide/02-building-open5gs-from-sources/
- UERANSIM v3.2.6 (2023.03.17) - https://github.com/aligungr/UERANSIM/wiki/Installation

Install MongoDB on Open5GS 5GC C-Plane machine.
It is not necessary to install MongoDB on Open5GS 5GC U-Plane machines.
[MongoDB Compass](https://www.mongodb.com/products/compass) is a convenient tool to look at the MongoDB database.

<h2 id="run">Run Open5GS 5GC and UERANSIM UE / RAN</h2>

First run the 5GC, then UERANSIM (UE & RAN implementation).

<h3 id="run_cp">Run Open5GS 5GC C-Plane</h3>

First, run Open5GS 5GC C-Plane.

- Open5GS 5GC C-Plane
```
./install/bin/open5gs-nrfd &
sleep 5
./install/bin/open5gs-scpd &
sleep 5
./install/bin/open5gs-smfd -c install/etc/open5gs/smf1.yaml &
./install/bin/open5gs-smfd -c install/etc/open5gs/smf2.yaml &
./install/bin/open5gs-amfd &
./install/bin/open5gs-ausfd &
./install/bin/open5gs-udmd &
./install/bin/open5gs-udrd &
./install/bin/open5gs-pcfd &
./install/bin/open5gs-nssfd &
./install/bin/open5gs-bsfd &
```

<h3 id="run_up">Run Open5GS 5GC U-Plane1 & U-Plane2</h3>

Next, run Open5GS 5GC U-Plane.

- Open5GS 5GC U-Plane1
```
./install/bin/open5gs-upfd &
```
- Open5GS 5GC U-Plane2
```
./install/bin/open5gs-upfd &
```
Then run `tcpdump` on one more terminal for each U-Plane.
- Run `tcpdump` on VM2 (U-Plane1)
```
# tcpdump -i ogstun -n
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on ogstun, link-type RAW (Raw IP), capture size 262144 bytes
```
- Run `tcpdump` on VM3 (U-Plane2)
```
# tcpdump -i ogstun -n
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on ogstun, link-type RAW (Raw IP), capture size 262144 bytes
```

<h3 id="run_ran">Run UERANSIM (gNodeB)</h3>

Please refer to the following for usage of UERANSIM.

https://github.com/aligungr/UERANSIM/wiki/Usage

```
# ./nr-gnb -c ../config/open5gs-gnb.yaml
UERANSIM v3.2.6
[2023-03-17 23:31:53.799] [sctp] [info] Trying to establish SCTP connection... (192.168.0.111:38412)
[2023-03-17 23:31:53.802] [sctp] [info] SCTP connection established (192.168.0.111:38412)
[2023-03-17 23:31:53.802] [sctp] [debug] SCTP association setup ascId[5]
[2023-03-17 23:31:53.802] [ngap] [debug] Sending NG Setup Request
[2023-03-17 23:31:53.803] [ngap] [debug] NG Setup Response received
[2023-03-17 23:31:53.803] [ngap] [info] NG Setup procedure is successful
```
The Open5GS C-Plane log when executed is as follows.
```
03/17 23:31:53.809: [amf] INFO: gNB-N2 accepted[192.168.0.131]:57846 in ng-path module (../src/amf/ngap-sctp.c:113)
03/17 23:31:53.809: [amf] INFO: gNB-N2 accepted[192.168.0.131] in master_sm module (../src/amf/amf-sm.c:733)
03/17 23:31:53.810: [amf] INFO: [Added] Number of gNBs is now 1 (../src/amf/context.c:1034)
03/17 23:31:53.810: [amf] INFO: gNB-N2[192.168.0.131] max_num_of_ostreams : 10 (../src/amf/amf-sm.c:772)
```

<h3 id="run_sd1">Run UERANSIM (UE[SST:1, SD:0x000001] connects to U-Plane1)</h3>

UE connects to U-Plane1 based on SST:1 and SD:0x000001.

```
# ./nr-ue -c ../config/open5gs-ue-sd1.yaml 
UERANSIM v3.2.6
[2023-03-17 23:32:36.622] [nas] [info] UE switches to state [MM-DEREGISTERED/PLMN-SEARCH]
[2023-03-17 23:32:36.622] [rrc] [debug] New signal detected for cell[1], total [1] cells in coverage
[2023-03-17 23:32:36.623] [nas] [info] Selected plmn[001/01]
[2023-03-17 23:32:36.623] [rrc] [info] Selected cell plmn[001/01] tac[1] category[SUITABLE]
[2023-03-17 23:32:36.623] [nas] [info] UE switches to state [MM-DEREGISTERED/PS]
[2023-03-17 23:32:36.623] [nas] [info] UE switches to state [MM-DEREGISTERED/NORMAL-SERVICE]
[2023-03-17 23:32:36.623] [nas] [debug] Initial registration required due to [MM-DEREG-NORMAL-SERVICE]
[2023-03-17 23:32:36.624] [nas] [debug] UAC access attempt is allowed for identity[0], category[MO_sig]
[2023-03-17 23:32:36.624] [nas] [debug] Sending Initial Registration
[2023-03-17 23:32:36.624] [nas] [info] UE switches to state [MM-REGISTER-INITIATED]
[2023-03-17 23:32:36.624] [rrc] [debug] Sending RRC Setup Request
[2023-03-17 23:32:36.625] [rrc] [info] RRC connection established
[2023-03-17 23:32:36.625] [rrc] [info] UE switches to state [RRC-CONNECTED]
[2023-03-17 23:32:36.625] [nas] [info] UE switches to state [CM-CONNECTED]
[2023-03-17 23:32:36.636] [nas] [debug] Authentication Request received
[2023-03-17 23:32:36.641] [nas] [debug] Security Mode Command received
[2023-03-17 23:32:36.641] [nas] [debug] Selected integrity[2] ciphering[0]
[2023-03-17 23:32:36.663] [nas] [debug] Registration accept received
[2023-03-17 23:32:36.663] [nas] [info] UE switches to state [MM-REGISTERED/NORMAL-SERVICE]
[2023-03-17 23:32:36.664] [nas] [debug] Sending Registration Complete
[2023-03-17 23:32:36.664] [nas] [info] Initial Registration is successful
[2023-03-17 23:32:36.664] [nas] [debug] Sending PDU Session Establishment Request
[2023-03-17 23:32:36.665] [nas] [debug] UAC access attempt is allowed for identity[0], category[MO_sig]
[2023-03-17 23:32:36.874] [nas] [debug] Configuration Update Command received
[2023-03-17 23:32:36.897] [nas] [debug] PDU Session Establishment Accept received
[2023-03-17 23:32:36.901] [nas] [info] PDU Session establishment is successful PSI[1]
[2023-03-17 23:32:36.927] [app] [info] Connection setup for PDU session[1] is successful, TUN interface[uesimtun0, 10.45.0.2] is up.
```
The Open5GS C-Plane log when executed is as follows.
```
03/17 23:32:36.627: [amf] INFO: InitialUEMessage (../src/amf/ngap-handler.c:372)
03/17 23:32:36.627: [amf] INFO: [Added] Number of gNB-UEs is now 1 (../src/amf/context.c:2327)
03/17 23:32:36.627: [amf] INFO:     RAN_UE_NGAP_ID[1] AMF_UE_NGAP_ID[1] TAC[1] CellID[0x10] (../src/amf/ngap-handler.c:533)
03/17 23:32:36.628: [amf] INFO: [suci-0-001-01-0001-0-0-0000000000] Unknown UE by SUCI (../src/amf/context.c:1634)
03/17 23:32:36.628: [amf] INFO: [Added] Number of AMF-UEs is now 1 (../src/amf/context.c:1419)
03/17 23:32:36.628: [gmm] INFO: Registration request (../src/amf/gmm-sm.c:985)
03/17 23:32:36.628: [gmm] INFO: [suci-0-001-01-0001-0-0-0000000000]    SUCI (../src/amf/gmm-handler.c:149)
03/17 23:32:36.629: [sbi] WARNING: [c43fc724-c4ca-41ed-b2da-fff6b71e2f9d] (NF-discover) NF has already been added (../lib/sbi/nnrf-handler.c:839)
03/17 23:32:36.629: [sbi] WARNING: NF EndPoint updated [127.0.0.11:80] (../lib/sbi/context.c:1618)
03/17 23:32:36.630: [sbi] WARNING: NF EndPoint updated [127.0.0.11:7777] (../lib/sbi/context.c:1527)
03/17 23:32:36.630: [sbi] INFO: [c43fc724-c4ca-41ed-b2da-fff6b71e2f9d] (NF-discover) NF Profile updated (../lib/sbi/nnrf-handler.c:862)
03/17 23:32:36.631: [sbi] WARNING: [c440159e-c4ca-41ed-bdda-e794d93e99c1] (NF-discover) NF has already been added (../lib/sbi/nnrf-handler.c:839)
03/17 23:32:36.631: [sbi] WARNING: NF EndPoint updated [127.0.0.12:80] (../lib/sbi/context.c:1618)
03/17 23:32:36.632: [sbi] WARNING: NF EndPoint updated [127.0.0.12:7777] (../lib/sbi/context.c:1527)
03/17 23:32:36.632: [sbi] WARNING: NF EndPoint updated [127.0.0.12:7777] (../lib/sbi/context.c:1527)
03/17 23:32:36.632: [sbi] WARNING: NF EndPoint updated [127.0.0.12:7777] (../lib/sbi/context.c:1527)
03/17 23:32:36.632: [sbi] INFO: [c440159e-c4ca-41ed-bdda-e794d93e99c1] (NF-discover) NF Profile updated (../lib/sbi/nnrf-handler.c:862)
03/17 23:32:36.644: [sbi] WARNING: [c440159e-c4ca-41ed-bdda-e794d93e99c1] (NF-discover) NF has already been added (../lib/sbi/nnrf-handler.c:839)
03/17 23:32:36.645: [sbi] WARNING: NF EndPoint updated [127.0.0.12:80] (../lib/sbi/context.c:1618)
03/17 23:32:36.645: [sbi] WARNING: NF EndPoint updated [127.0.0.12:7777] (../lib/sbi/context.c:1527)
03/17 23:32:36.645: [sbi] WARNING: NF EndPoint updated [127.0.0.12:7777] (../lib/sbi/context.c:1527)
03/17 23:32:36.645: [sbi] WARNING: NF EndPoint updated [127.0.0.12:7777] (../lib/sbi/context.c:1527)
03/17 23:32:36.645: [sbi] INFO: [c440159e-c4ca-41ed-bdda-e794d93e99c1] (NF-discover) NF Profile updated (../lib/sbi/nnrf-handler.c:862)
03/17 23:32:36.647: [sbi] WARNING: [c440159e-c4ca-41ed-bdda-e794d93e99c1] (NF-discover) NF has already been added (../lib/sbi/nnrf-handler.c:839)
03/17 23:32:36.648: [sbi] WARNING: NF EndPoint updated [127.0.0.12:80] (../lib/sbi/context.c:1618)
03/17 23:32:36.648: [sbi] WARNING: NF EndPoint updated [127.0.0.12:7777] (../lib/sbi/context.c:1527)
03/17 23:32:36.648: [sbi] WARNING: NF EndPoint updated [127.0.0.12:7777] (../lib/sbi/context.c:1527)
03/17 23:32:36.648: [sbi] WARNING: NF EndPoint updated [127.0.0.12:7777] (../lib/sbi/context.c:1527)
03/17 23:32:36.648: [sbi] INFO: [c440159e-c4ca-41ed-bdda-e794d93e99c1] (NF-discover) NF Profile updated (../lib/sbi/nnrf-handler.c:862)
03/17 23:32:36.655: [sbi] WARNING: [c448b5e6-c4ca-41ed-93bd-e5b96bc2423f] (NF-discover) NF has already been added (../lib/sbi/nnrf-handler.c:839)
03/17 23:32:36.656: [sbi] WARNING: NF EndPoint updated [127.0.0.13:80] (../lib/sbi/context.c:1618)
03/17 23:32:36.656: [sbi] WARNING: NF EndPoint updated [127.0.0.13:7777] (../lib/sbi/context.c:1527)
03/17 23:32:36.656: [sbi] WARNING: NF EndPoint updated [127.0.0.13:7777] (../lib/sbi/context.c:1527)
03/17 23:32:36.656: [sbi] WARNING: NF EndPoint updated [127.0.0.13:7777] (../lib/sbi/context.c:1527)
03/17 23:32:36.656: [sbi] INFO: [c448b5e6-c4ca-41ed-93bd-e5b96bc2423f] (NF-discover) NF Profile updated (../lib/sbi/nnrf-handler.c:862)
03/17 23:32:36.660: [sbi] WARNING: [c448791e-c4ca-41ed-a9c0-538da6018bde] (NF-discover) NF has already been added (../lib/sbi/nnrf-handler.c:839)
03/17 23:32:36.660: [sbi] WARNING: NF EndPoint updated [127.0.0.20:80] (../lib/sbi/context.c:1618)
03/17 23:32:36.660: [sbi] WARNING: NF EndPoint updated [127.0.0.20:7777] (../lib/sbi/context.c:1527)
03/17 23:32:36.661: [sbi] INFO: [c448791e-c4ca-41ed-a9c0-538da6018bde] (NF-discover) NF Profile updated (../lib/sbi/nnrf-handler.c:862)
03/17 23:32:36.872: [gmm] INFO: [imsi-001010000000000] Registration complete (../src/amf/gmm-sm.c:1917)
03/17 23:32:36.873: [amf] INFO: [imsi-001010000000000] Configuration update command (../src/amf/nas-path.c:612)
03/17 23:32:36.873: [gmm] INFO:     UTC [2023-03-17T14:32:36] Timezone[0]/DST[0] (../src/amf/gmm-build.c:545)
03/17 23:32:36.874: [gmm] INFO:     LOCAL [2023-03-17T23:32:36] Timezone[32400]/DST[0] (../src/amf/gmm-build.c:550)
03/17 23:32:36.875: [amf] INFO: [Added] Number of AMF-Sessions is now 1 (../src/amf/context.c:2348)
03/17 23:32:36.875: [gmm] INFO: UE SUPI[imsi-001010000000000] DNN[internet] S_NSSAI[SST:1 SD:0x1] (../src/amf/gmm-handler.c:1186)
03/17 23:32:36.878: [smf] INFO: [Added] Number of SMF-UEs is now 1 (../src/smf/context.c:1012)
03/17 23:32:36.878: [smf] INFO: [Added] Number of SMF-Sessions is now 1 (../src/smf/context.c:3108)
03/17 23:32:36.881: [sbi] WARNING: [c440159e-c4ca-41ed-bdda-e794d93e99c1] (NF-discover) NF has already been added (../lib/sbi/nnrf-handler.c:839)
03/17 23:32:36.881: [sbi] WARNING: NF EndPoint updated [127.0.0.12:80] (../lib/sbi/context.c:1618)
03/17 23:32:36.881: [sbi] WARNING: NF EndPoint updated [127.0.0.12:7777] (../lib/sbi/context.c:1527)
03/17 23:32:36.881: [sbi] WARNING: NF EndPoint updated [127.0.0.12:7777] (../lib/sbi/context.c:1527)
03/17 23:32:36.882: [sbi] WARNING: NF EndPoint updated [127.0.0.12:7777] (../lib/sbi/context.c:1527)
03/17 23:32:36.882: [sbi] INFO: [c440159e-c4ca-41ed-bdda-e794d93e99c1] (NF-discover) NF Profile updated (../lib/sbi/nnrf-handler.c:862)
03/17 23:32:36.887: [sbi] WARNING: [c448b5e6-c4ca-41ed-93bd-e5b96bc2423f] (NF-discover) NF has already been added (../lib/sbi/nnrf-handler.c:839)
03/17 23:32:36.887: [sbi] WARNING: NF EndPoint updated [127.0.0.13:80] (../lib/sbi/context.c:1618)
03/17 23:32:36.888: [sbi] WARNING: NF EndPoint updated [127.0.0.13:7777] (../lib/sbi/context.c:1527)
03/17 23:32:36.888: [sbi] WARNING: NF EndPoint updated [127.0.0.13:7777] (../lib/sbi/context.c:1527)
03/17 23:32:36.888: [sbi] WARNING: NF EndPoint updated [127.0.0.13:7777] (../lib/sbi/context.c:1527)
03/17 23:32:36.888: [sbi] INFO: [c448b5e6-c4ca-41ed-93bd-e5b96bc2423f] (NF-discover) NF Profile updated (../lib/sbi/nnrf-handler.c:862)
03/17 23:32:36.889: [sbi] WARNING: [c448791e-c4ca-41ed-a9c0-538da6018bde] (NF-discover) NF has already been added (../lib/sbi/nnrf-handler.c:839)
03/17 23:32:36.890: [sbi] WARNING: NF EndPoint updated [127.0.0.20:80] (../lib/sbi/context.c:1618)
03/17 23:32:36.890: [sbi] WARNING: NF EndPoint updated [127.0.0.20:7777] (../lib/sbi/context.c:1527)
03/17 23:32:36.890: [sbi] INFO: [c448791e-c4ca-41ed-a9c0-538da6018bde] (NF-discover) NF Profile updated (../lib/sbi/nnrf-handler.c:862)
03/17 23:32:36.892: [sbi] WARNING: [c43f844e-c4ca-41ed-8cfd-93480d9580f6] (NF-discover) NF has already been added (../lib/sbi/nnrf-handler.c:839)
03/17 23:32:36.892: [sbi] WARNING: NF EndPoint updated [127.0.0.15:80] (../lib/sbi/context.c:1618)
03/17 23:32:36.892: [sbi] WARNING: NF EndPoint updated [127.0.0.15:7777] (../lib/sbi/context.c:1527)
03/17 23:32:36.892: [sbi] INFO: [c43f844e-c4ca-41ed-8cfd-93480d9580f6] (NF-discover) NF Profile updated (../lib/sbi/nnrf-handler.c:862)
03/17 23:32:36.894: [smf] INFO: UE SUPI[imsi-001010000000000] DNN[internet] IPv4[10.45.0.2] IPv6[] (../src/smf/npcf-handler.c:528)
03/17 23:32:36.895: [gtp] INFO: gtp_connect() [192.168.0.114]:2152 (../lib/gtp/path.c:60)
```
The Open5GS U-Plane1 log when executed is as follows.
```
03/17 23:32:36.887: [upf] INFO: [Added] Number of UPF-Sessions is now 1 (../src/upf/context.c:194)
03/17 23:32:36.887: [gtp] INFO: gtp_connect() [192.168.0.112]:2152 (../lib/gtp/path.c:60)
03/17 23:32:36.887: [upf] INFO: UE F-SEID[UP:0x1 CP:0x1] APN[internet] PDN-Type[1] IPv4[10.45.0.2] IPv6[] (../src/upf/context.c:467)
03/17 23:32:36.887: [upf] INFO: UE F-SEID[UP:0x1 CP:0x1] APN[internet] PDN-Type[1] IPv4[10.45.0.2] IPv6[] (../src/upf/context.c:467)
03/17 23:32:36.892: [gtp] INFO: gtp_connect() [192.168.0.131]:2152 (../lib/gtp/path.c:60)
```
The TUNnel interface `uesimtun0` is created as follows.
```
# ip addr show
...
33: uesimtun0: <POINTOPOINT,PROMISC,NOTRAILERS,UP,LOWER_UP> mtu 1400 qdisc fq_codel state UNKNOWN group default qlen 500
    link/none 
    inet 10.45.0.2/32 scope global uesimtun0
       valid_lft forever preferred_lft forever
    inet6 fe80::98ce:51e5:4987:7761/64 scope link stable-privacy 
       valid_lft forever preferred_lft forever
...
```

<h3 id="run_sd2">Run UERANSIM (UE[SST:1, SD:0x000002] connects to U-Plane2)</h3>

Additionally, UE connects to U-Plane2 based on SST:1 and SD:0x000002.

```
# ./nr-ue -c ../config/open5gs-ue-sd2.yaml 
UERANSIM v3.2.6
[2023-03-18 00:06:49.069] [nas] [info] UE switches to state [MM-DEREGISTERED/PLMN-SEARCH]
[2023-03-18 00:06:49.070] [rrc] [debug] New signal detected for cell[1], total [1] cells in coverage
[2023-03-18 00:06:49.070] [nas] [info] Selected plmn[001/01]
[2023-03-18 00:06:49.071] [rrc] [info] Selected cell plmn[001/01] tac[1] category[SUITABLE]
[2023-03-18 00:06:49.071] [nas] [info] UE switches to state [MM-DEREGISTERED/PS]
[2023-03-18 00:06:49.071] [nas] [info] UE switches to state [MM-DEREGISTERED/NORMAL-SERVICE]
[2023-03-18 00:06:49.071] [nas] [debug] Initial registration required due to [MM-DEREG-NORMAL-SERVICE]
[2023-03-18 00:06:49.073] [nas] [debug] UAC access attempt is allowed for identity[0], category[MO_sig]
[2023-03-18 00:06:49.073] [nas] [debug] Sending Initial Registration
[2023-03-18 00:06:49.074] [rrc] [debug] Sending RRC Setup Request
[2023-03-18 00:06:49.074] [nas] [info] UE switches to state [MM-REGISTER-INITIATED]
[2023-03-18 00:06:49.074] [rrc] [info] RRC connection established
[2023-03-18 00:06:49.075] [rrc] [info] UE switches to state [RRC-CONNECTED]
[2023-03-18 00:06:49.075] [nas] [info] UE switches to state [CM-CONNECTED]
[2023-03-18 00:06:49.089] [nas] [debug] Authentication Request received
[2023-03-18 00:06:49.093] [nas] [debug] Security Mode Command received
[2023-03-18 00:06:49.093] [nas] [debug] Selected integrity[2] ciphering[0]
[2023-03-18 00:06:49.113] [nas] [debug] Registration accept received
[2023-03-18 00:06:49.113] [nas] [info] UE switches to state [MM-REGISTERED/NORMAL-SERVICE]
[2023-03-18 00:06:49.113] [nas] [debug] Sending Registration Complete
[2023-03-18 00:06:49.113] [nas] [info] Initial Registration is successful
[2023-03-18 00:06:49.114] [nas] [debug] Sending PDU Session Establishment Request
[2023-03-18 00:06:49.114] [nas] [debug] UAC access attempt is allowed for identity[0], category[MO_sig]
[2023-03-18 00:06:49.325] [nas] [debug] Configuration Update Command received
[2023-03-18 00:06:49.345] [nas] [debug] PDU Session Establishment Accept received
[2023-03-18 00:06:49.351] [nas] [info] PDU Session establishment is successful PSI[1]
[2023-03-18 00:06:49.375] [app] [info] Connection setup for PDU session[1] is successful, TUN interface[uesimtun1, 10.46.0.2] is up.
```
The Open5GS C-Plane log when executed is as follows.
```
03/18 00:06:49.074: [amf] INFO: InitialUEMessage (../src/amf/ngap-handler.c:372)
03/18 00:06:49.075: [amf] INFO: [Added] Number of gNB-UEs is now 2 (../src/amf/context.c:2327)
03/18 00:06:49.075: [amf] INFO:     RAN_UE_NGAP_ID[2] AMF_UE_NGAP_ID[2] TAC[1] CellID[0x10] (../src/amf/ngap-handler.c:533)
03/18 00:06:49.075: [amf] INFO: [suci-0-001-01-0002-0-0-0000000000] Unknown UE by SUCI (../src/amf/context.c:1634)
03/18 00:06:49.075: [amf] INFO: [Added] Number of AMF-UEs is now 2 (../src/amf/context.c:1419)
03/18 00:06:49.075: [gmm] INFO: Registration request (../src/amf/gmm-sm.c:985)
03/18 00:06:49.075: [gmm] INFO: [suci-0-001-01-0002-0-0-0000000000]    SUCI (../src/amf/gmm-handler.c:149)
03/18 00:06:49.077: [sbi] WARNING: [c43fc724-c4ca-41ed-b2da-fff6b71e2f9d] (NF-discover) NF has already been added (../lib/sbi/nnrf-handler.c:839)
03/18 00:06:49.077: [sbi] WARNING: NF EndPoint updated [127.0.0.11:80] (../lib/sbi/context.c:1618)
03/18 00:06:49.077: [sbi] WARNING: NF EndPoint updated [127.0.0.11:7777] (../lib/sbi/context.c:1527)
03/18 00:06:49.077: [sbi] INFO: [c43fc724-c4ca-41ed-b2da-fff6b71e2f9d] (NF-discover) NF Profile updated (../lib/sbi/nnrf-handler.c:862)
03/18 00:06:49.080: [sbi] WARNING: [c440159e-c4ca-41ed-bdda-e794d93e99c1] (NF-discover) NF has already been added (../lib/sbi/nnrf-handler.c:839)
03/18 00:06:49.081: [sbi] WARNING: NF EndPoint updated [127.0.0.12:80] (../lib/sbi/context.c:1618)
03/18 00:06:49.081: [sbi] WARNING: NF EndPoint updated [127.0.0.12:7777] (../lib/sbi/context.c:1527)
03/18 00:06:49.081: [sbi] WARNING: NF EndPoint updated [127.0.0.12:7777] (../lib/sbi/context.c:1527)
03/18 00:06:49.081: [sbi] WARNING: NF EndPoint updated [127.0.0.12:7777] (../lib/sbi/context.c:1527)
03/18 00:06:49.082: [sbi] INFO: [c440159e-c4ca-41ed-bdda-e794d93e99c1] (NF-discover) NF Profile updated (../lib/sbi/nnrf-handler.c:862)
03/18 00:06:49.094: [sbi] WARNING: [c440159e-c4ca-41ed-bdda-e794d93e99c1] (NF-discover) NF has already been added (../lib/sbi/nnrf-handler.c:839)
03/18 00:06:49.094: [sbi] WARNING: NF EndPoint updated [127.0.0.12:80] (../lib/sbi/context.c:1618)
03/18 00:06:49.094: [sbi] WARNING: NF EndPoint updated [127.0.0.12:7777] (../lib/sbi/context.c:1527)
03/18 00:06:49.094: [sbi] WARNING: NF EndPoint updated [127.0.0.12:7777] (../lib/sbi/context.c:1527)
03/18 00:06:49.094: [sbi] WARNING: NF EndPoint updated [127.0.0.12:7777] (../lib/sbi/context.c:1527)
03/18 00:06:49.095: [sbi] INFO: [c440159e-c4ca-41ed-bdda-e794d93e99c1] (NF-discover) NF Profile updated (../lib/sbi/nnrf-handler.c:862)
03/18 00:06:49.097: [sbi] WARNING: [c440159e-c4ca-41ed-bdda-e794d93e99c1] (NF-discover) NF has already been added (../lib/sbi/nnrf-handler.c:839)
03/18 00:06:49.097: [sbi] WARNING: NF EndPoint updated [127.0.0.12:80] (../lib/sbi/context.c:1618)
03/18 00:06:49.097: [sbi] WARNING: NF EndPoint updated [127.0.0.12:7777] (../lib/sbi/context.c:1527)
03/18 00:06:49.097: [sbi] WARNING: NF EndPoint updated [127.0.0.12:7777] (../lib/sbi/context.c:1527)
03/18 00:06:49.097: [sbi] WARNING: NF EndPoint updated [127.0.0.12:7777] (../lib/sbi/context.c:1527)
03/18 00:06:49.098: [sbi] INFO: [c440159e-c4ca-41ed-bdda-e794d93e99c1] (NF-discover) NF Profile updated (../lib/sbi/nnrf-handler.c:862)
03/18 00:06:49.106: [sbi] WARNING: [c448b5e6-c4ca-41ed-93bd-e5b96bc2423f] (NF-discover) NF has already been added (../lib/sbi/nnrf-handler.c:839)
03/18 00:06:49.106: [sbi] WARNING: NF EndPoint updated [127.0.0.13:80] (../lib/sbi/context.c:1618)
03/18 00:06:49.106: [sbi] WARNING: NF EndPoint updated [127.0.0.13:7777] (../lib/sbi/context.c:1527)
03/18 00:06:49.106: [sbi] WARNING: NF EndPoint updated [127.0.0.13:7777] (../lib/sbi/context.c:1527)
03/18 00:06:49.107: [sbi] WARNING: NF EndPoint updated [127.0.0.13:7777] (../lib/sbi/context.c:1527)
03/18 00:06:49.107: [sbi] INFO: [c448b5e6-c4ca-41ed-93bd-e5b96bc2423f] (NF-discover) NF Profile updated (../lib/sbi/nnrf-handler.c:862)
03/18 00:06:49.108: [pcf] WARNING: NF EndPoint updated [127.0.0.5:7777] (../src/pcf/npcf-handler.c:100)
03/18 00:06:49.321: [gmm] INFO: [imsi-001010000000000] Registration complete (../src/amf/gmm-sm.c:1917)
03/18 00:06:49.321: [amf] INFO: [imsi-001010000000000] Configuration update command (../src/amf/nas-path.c:612)
03/18 00:06:49.322: [gmm] INFO:     UTC [2023-03-17T15:06:49] Timezone[0]/DST[0] (../src/amf/gmm-build.c:545)
03/18 00:06:49.322: [gmm] INFO:     LOCAL [2023-03-18T00:06:49] Timezone[32400]/DST[0] (../src/amf/gmm-build.c:550)
03/18 00:06:49.323: [amf] INFO: [Added] Number of AMF-Sessions is now 2 (../src/amf/context.c:2348)
03/18 00:06:49.324: [gmm] INFO: UE SUPI[imsi-001010000000000] DNN[internet] S_NSSAI[SST:1 SD:0x2] (../src/amf/gmm-handler.c:1186)
03/18 00:06:49.327: [smf] INFO: [Added] Number of SMF-UEs is now 1 (../src/smf/context.c:1012)
03/18 00:06:49.327: [smf] INFO: [Added] Number of SMF-Sessions is now 1 (../src/smf/context.c:3108)
03/18 00:06:49.330: [sbi] WARNING: [c440159e-c4ca-41ed-bdda-e794d93e99c1] (NF-discover) NF has already been added (../lib/sbi/nnrf-handler.c:839)
03/18 00:06:49.330: [sbi] WARNING: NF EndPoint updated [127.0.0.12:80] (../lib/sbi/context.c:1618)
03/18 00:06:49.330: [sbi] WARNING: NF EndPoint updated [127.0.0.12:7777] (../lib/sbi/context.c:1527)
03/18 00:06:49.330: [sbi] WARNING: NF EndPoint updated [127.0.0.12:7777] (../lib/sbi/context.c:1527)
03/18 00:06:49.331: [sbi] WARNING: NF EndPoint updated [127.0.0.12:7777] (../lib/sbi/context.c:1527)
03/18 00:06:49.331: [sbi] INFO: [c440159e-c4ca-41ed-bdda-e794d93e99c1] (NF-discover) NF Profile updated (../lib/sbi/nnrf-handler.c:862)
03/18 00:06:49.335: [sbi] WARNING: [c448b5e6-c4ca-41ed-93bd-e5b96bc2423f] (NF-discover) NF has already been added (../lib/sbi/nnrf-handler.c:839)
03/18 00:06:49.335: [sbi] WARNING: NF EndPoint updated [127.0.0.13:80] (../lib/sbi/context.c:1618)
03/18 00:06:49.335: [sbi] WARNING: NF EndPoint updated [127.0.0.13:7777] (../lib/sbi/context.c:1527)
03/18 00:06:49.335: [sbi] WARNING: NF EndPoint updated [127.0.0.13:7777] (../lib/sbi/context.c:1527)
03/18 00:06:49.335: [sbi] WARNING: NF EndPoint updated [127.0.0.13:7777] (../lib/sbi/context.c:1527)
03/18 00:06:49.335: [sbi] INFO: [c448b5e6-c4ca-41ed-93bd-e5b96bc2423f] (NF-discover) NF Profile updated (../lib/sbi/nnrf-handler.c:862)
03/18 00:06:49.336: [pcf] WARNING: NF EndPoint updated [127.0.0.4:7777] (../src/pcf/npcf-handler.c:291)
03/18 00:06:49.340: [smf] INFO: UE SUPI[imsi-001010000000000] DNN[internet] IPv4[10.46.0.2] IPv6[] (../src/smf/npcf-handler.c:528)
03/18 00:06:49.341: [gtp] INFO: gtp_connect() [192.168.0.115]:2152 (../lib/gtp/path.c:60)
```
The Open5GS U-Plane2 log when executed is as follows.
```
03/18 00:06:49.330: [upf] INFO: [Added] Number of UPF-Sessions is now 1 (../src/upf/context.c:194)
03/18 00:06:49.330: [gtp] INFO: gtp_connect() [192.168.0.113]:2152 (../lib/gtp/path.c:60)
03/18 00:06:49.330: [upf] INFO: UE F-SEID[UP:0x1 CP:0x1] APN[internet] PDN-Type[1] IPv4[10.46.0.2] IPv6[] (../src/upf/context.c:467)
03/18 00:06:49.330: [upf] INFO: UE F-SEID[UP:0x1 CP:0x1] APN[internet] PDN-Type[1] IPv4[10.46.0.2] IPv6[] (../src/upf/context.c:467)
03/18 00:06:49.335: [gtp] INFO: gtp_connect() [192.168.0.131]:2152 (../lib/gtp/path.c:60)
```
The TUNnel interface `uesimtun1` is created as follows.
```
# ip addr show
...
34: uesimtun1: <POINTOPOINT,PROMISC,NOTRAILERS,UP,LOWER_UP> mtu 1400 qdisc fq_codel state UNKNOWN group default qlen 500
    link/none 
    inet 10.46.0.2/32 scope global uesimtun1
       valid_lft forever preferred_lft forever
    inet6 fe80::dbf8:6216:feb8:2d5d/64 scope link stable-privacy 
       valid_lft forever preferred_lft forever
...
```

<h3 id="ping_sd1">Ping google.com going through DN=10.45.0.0/16 on U-Plane1 via uesimtun0</h3>

Confirm by using `tcpdump` that the packet goes through `if=ogstun` on U-Plane1.
```
# ping google.com -I uesimtun0 -n
PING google.com (172.217.26.238) from 10.45.0.2 uesimtun0: 56(84) bytes of data.
64 bytes from 172.217.26.238: icmp_seq=1 ttl=61 time=20.0 ms
64 bytes from 172.217.26.238: icmp_seq=2 ttl=61 time=18.5 ms
64 bytes from 172.217.26.238: icmp_seq=3 ttl=61 time=18.4 ms
```
The `tcpdump` log on U-Plane1 is as follows.
```
00:13:29.957030 IP 10.45.0.2 > 172.217.26.238: ICMP echo request, id 50, seq 1, length 64
00:13:29.974555 IP 172.217.26.238 > 10.45.0.2: ICMP echo reply, id 50, seq 1, length 64
00:13:30.957574 IP 10.45.0.2 > 172.217.26.238: ICMP echo request, id 50, seq 2, length 64
00:13:30.973632 IP 172.217.26.238 > 10.45.0.2: ICMP echo reply, id 50, seq 2, length 64
00:13:31.958465 IP 10.45.0.2 > 172.217.26.238: ICMP echo request, id 50, seq 3, length 64
00:13:31.974465 IP 172.217.26.238 > 10.45.0.2: ICMP echo reply, id 50, seq 3, length 64
```
**Note. Make sure the packet does not go through U-Plane2.**

<h3 id="ping_sd2">Ping google.com going through DN=10.46.0.0/16 on U-Plane2 via uesimtun1</h3>

Confirm by using `tcpdump` that the packet goes through `if=ogstun` on U-Plane2.
```
# ping google.com -I uesimtun1 -n
PING google.com (172.217.26.238) from 10.46.0.2 uesimtun1: 56(84) bytes of data.
64 bytes from 172.217.26.238: icmp_seq=1 ttl=61 time=20.2 ms
64 bytes from 172.217.26.238: icmp_seq=2 ttl=61 time=17.7 ms
64 bytes from 172.217.26.238: icmp_seq=3 ttl=61 time=18.3 ms
```
The `tcpdump` log on U-Plane2 is as follows.
```
00:15:18.617823 IP 172.217.26.238 > 10.46.0.2: ICMP echo reply, id 51, seq 1, length 64
00:15:19.600942 IP 10.46.0.2 > 172.217.26.238: ICMP echo request, id 51, seq 2, length 64
00:15:19.616712 IP 172.217.26.238 > 10.46.0.2: ICMP echo reply, id 51, seq 2, length 64
00:15:20.602197 IP 10.46.0.2 > 172.217.26.238: ICMP echo request, id 51, seq 3, length 64
00:15:20.618310 IP 172.217.26.238 > 10.46.0.2: ICMP echo reply, id 51, seq 3, length 64
```
**Note. Make sure the packet does not go through U-Plane1.**

---
I was able to confirm the very simple configuration in which one UE connects to the UPF based on S-NSSAI. I would like to thank the excellent developers and all the contributors of Open5GS and UERANSIM.

<h2 id="changelog">Changelog (summary)</h2>

- [2023.03.17] Updated to Open5GS v2.6.1 (2023.03.17) and UERANSIM v3.2.6 (2023.03.17).
- [2023.01.13] Updated to Open5GS v2.5.6.
- [2022.08.01] Initial release.
