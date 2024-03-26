# Open5GS 5GC & UERANSIM UE / RAN Sample Configuration - Select UPF based on S-NSSAI
This describes a very simple configuration that uses Open5GS and UERANSIM to select the UPF based on S-NSSAI.

---

### [Sample Configurations and Miscellaneous for Mobile Network](https://github.com/s5uishida/sample_config_misc_for_mobile_network)

---

<a id="toc"></a>

## Table of Contents

- [Overview of Open5GS 5GC Simulation Mobile Network](#overview)
- [Changes in configuration files of Open5GS 5GC and UERANSIM UE / RAN](#changes)
  - [Changes in configuration files of Open5GS 5GC C-Plane](#changes_cp)
  - [Changes in configuration files of Open5GS 5GC U-Plane1](#changes_up1)
  - [Changes in configuration files of Open5GS 5GC U-Plane2](#changes_up2)
  - [Changes in configuration files of UERANSIM UE / RAN](#changes_ueransim)
    - [Changes in configuration files of RAN (gNodeB)](#changes_ran)
    - [Changes in configuration files of UE[SST:1, SD:0x000001] (IMSI-001010000000000)](#changes_ue_sd1)
    - [Changes in configuration files of UE[SST:1, SD:0x000002] (IMSI-001010000000000)](#changes_ue_sd2)
- [Network settings of Open5GS 5GC and UERANSIM UE / RAN](#network_settings)
  - [Network settings of Open5GS 5GC C-Plane](#network_settings_cp)
  - [Network settings of Open5GS 5GC U-Plane1](#network_settings_up1)
  - [Network settings of Open5GS 5GC U-Plane2](#network_settings_up2)
- [Build Open5GS and UERANSIM](#build)
- [Run Open5GS 5GC and UERANSIM UE / RAN](#run)
  - [Run Open5GS 5GC C-Plane](#run_cp)
  - [Run Open5GS 5GC U-Plane1 & U-Plane2](#run_up)
  - [Run UERANSIM (gNodeB)](#run_ran)
  - [Run UERANSIM (UE[SST:1, SD:0x000001])](#run_sd1)
    - [UE connects to U-Plane1 based on SST:1 and SD:0x000001](#con_sd1)
    - [Ping google.com going through DN=10.45.0.0/16 on U-Plane1](#ping_sd1)
  - [Run UERANSIM (UE[SST:1, SD:0x000002])](#run_sd2)
    - [UE connects to U-Plane2 based on SST:1 and SD:0x000002](#con_sd2)
    - [Ping google.com going through DN=10.46.0.0/16 on U-Plane2](#ping_sd2)
- [Changelog (summary)](#changelog)

---
<a id="overview"></a>

## Overview of Open5GS 5GC Simulation Mobile Network

The following minimum configuration was set as a condition.
- The UE selects a pair of SMF and UPF based on S-NSSAI.

The built simulation environment is as follows.

<img src="./images/network-overview.png" title="./images/network-overview.png" width=1000px></img>

The 5GC / UE / RAN used are as follows.
- 5GC - Open5GS v2.7.0 (2024.03.24) - https://github.com/open5gs/open5gs
- UE / RAN - UERANSIM v3.2.6 (2024.03.08) - https://github.com/aligungr/UERANSIM

Each VMs are as follows.  
| VM # | SW & Role | IP address | OS | Memory (Min) | HDD (Min) |
| --- | --- | --- | --- | --- | --- |
| VM1 | Open5GS 5GC C-Plane | 192.168.0.111/24 <br> 192.168.0.112/24 <br> 192.168.0.113/24 | Ubuntu 22.04 | 2GB | 20GB |
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
| 10.46.0.0/16 | SST:1 <br> SD:0x000002 | ogstun | internet | uesimtun0 | U-Plane2 |

<a id="changes"></a>

## Changes in configuration files of Open5GS 5GC and UERANSIM UE / RAN

Please refer to the following for building Open5GS and UERANSIM respectively.
- Open5GS v2.7.0 (2024.03.24) - https://open5gs.org/open5gs/docs/guide/02-building-open5gs-from-sources/
- UERANSIM v3.2.6 (2024.03.08) - https://github.com/aligungr/UERANSIM/wiki/Installation

<a id="changes_cp"></a>

### Changes in configuration files of Open5GS 5GC C-Plane

- `open5gs/install/etc/open5gs/amf.yaml`
```diff
--- amf.yaml.orig       2024-03-24 15:36:48.000000000 +0900
+++ amf.yaml    2024-03-25 19:49:01.896848576 +0900
@@ -19,29 +19,32 @@
         - uri: http://127.0.0.200:7777
   ngap:
     server:
-      - address: 127.0.0.5
+      - address: 192.168.0.111
   metrics:
     server:
       - address: 127.0.0.5
         port: 9090
   guami:
     - plmn_id:
-        mcc: 999
-        mnc: 70
+        mcc: 001
+        mnc: 01
       amf_id:
         region: 2
         set: 1
   tai:
     - plmn_id:
-        mcc: 999
-        mnc: 70
+        mcc: 001
+        mnc: 01
       tac: 1
   plmn_support:
     - plmn_id:
-        mcc: 999
-        mnc: 70
+        mcc: 001
+        mnc: 01
       s_nssai:
         - sst: 1
+          sd: 000001
+        - sst: 1
+          sd: 000002
   security:
     integrity_order : [ NIA2, NIA1, NIA0 ]
     ciphering_order : [ NEA0, NEA1, NEA2 ]
```
- `open5gs/install/etc/open5gs/nrf.yaml`
```diff
--- nrf.yaml.orig       2024-03-24 15:36:48.000000000 +0900
+++ nrf.yaml    2024-03-25 19:46:56.184797762 +0900
@@ -10,8 +10,8 @@
 nrf:
   serving:  # 5G roaming requires PLMN in NRF
     - plmn_id:
-        mcc: 999
-        mnc: 70
+        mcc: 001
+        mnc: 01
   sbi:
     server:
       - address: 127.0.0.10
```
- `open5gs/install/etc/open5gs/nssf.yaml`
```diff
--- nssf.yaml.orig      2024-03-24 15:36:48.000000000 +0900
+++ nssf.yaml   2024-03-25 19:53:57.788260085 +0900
@@ -21,6 +21,11 @@
         - uri: http://127.0.0.10:7777
           s_nssai:
             sst: 1
+            sd: 000001
+        - uri: http://127.0.0.10:7777
+          s_nssai:
+            sst: 1
+            sd: 000002
 ################################################################################
 # SBI Server
 ################################################################################
```
- `open5gs/install/etc/open5gs/smf1.yaml`
```diff
--- smf.yaml.orig       2024-03-24 15:36:48.000000000 +0900
+++ smf1.yaml   2024-03-25 20:28:19.440826160 +0900
@@ -1,5 +1,5 @@
 logger:
-  file: /root/open5gs/install/var/log/open5gs/smf.log
+  file: /root/open5gs/install/var/log/open5gs/smf1.log
 #  level: info   # fatal|error|warn|info(default)|debug|trace
 
 global:
@@ -19,35 +19,45 @@
         - uri: http://127.0.0.200:7777
   pfcp:
     server:
-      - address: 127.0.0.4
+      - address: 192.168.0.112
     client:
       upf:
-        - address: 127.0.0.7
+        - address: 192.168.0.114
+          dnn: internet
   gtpc:
     server:
       - address: 127.0.0.4
   gtpu:
     server:
-      - address: 127.0.0.4
+      - address: 192.168.0.112
   metrics:
     server:
       - address: 127.0.0.4
         port: 9090
   session:
     - subnet: 10.45.0.1/16
-    - subnet: 2001:db8:cafe::1/48
+      dnn: internet
   dns:
     - 8.8.8.8
     - 8.8.4.4
-    - 2001:4860:4860::8888
-    - 2001:4860:4860::8844
   mtu: 1400
 #  p-cscf:
 #    - 127.0.0.1
 #    - ::1
 #  ctf:
 #    enabled: auto   # auto(default)|yes|no
-  freeDiameter: /root/open5gs/install/etc/freeDiameter/smf.conf
+#  freeDiameter: /root/open5gs/install/etc/freeDiameter/smf.conf
+  info:
+    - s_nssai:
+        - sst: 1
+          sd: 000001
+          dnn:
+            - internet
+      tai:
+        - plmn_id:
+            mcc: 001
+            mnc: 01
+          tac: 1
 
 ################################################################################
 # SMF Info
```
- `open5gs/install/etc/open5gs/smf2.yaml`
```diff
--- smf.yaml.orig       2024-03-24 15:36:48.000000000 +0900
+++ smf2.yaml   2024-03-25 20:28:41.255051446 +0900
@@ -1,5 +1,5 @@
 logger:
-  file: /root/open5gs/install/var/log/open5gs/smf.log
+  file: /root/open5gs/install/var/log/open5gs/smf2.log
 #  level: info   # fatal|error|warn|info(default)|debug|trace
 
 global:
@@ -10,7 +10,7 @@
 smf:
   sbi:
     server:
-      - address: 127.0.0.4
+      - address: 127.0.0.24
         port: 7777
     client:
 #      nrf:
@@ -19,35 +19,45 @@
         - uri: http://127.0.0.200:7777
   pfcp:
     server:
-      - address: 127.0.0.4
+      - address: 192.168.0.113
     client:
       upf:
-        - address: 127.0.0.7
+        - address: 192.168.0.115
+          dnn: internet
   gtpc:
     server:
-      - address: 127.0.0.4
+      - address: 127.0.0.24
   gtpu:
     server:
-      - address: 127.0.0.4
+      - address: 192.168.0.113
   metrics:
     server:
-      - address: 127.0.0.4
+      - address: 127.0.0.24
         port: 9090
   session:
-    - subnet: 10.45.0.1/16
-    - subnet: 2001:db8:cafe::1/48
+    - subnet: 10.46.0.1/16
+      dnn: internet
   dns:
     - 8.8.8.8
     - 8.8.4.4
-    - 2001:4860:4860::8888
-    - 2001:4860:4860::8844
   mtu: 1400
 #  p-cscf:
 #    - 127.0.0.1
 #    - ::1
 #  ctf:
 #    enabled: auto   # auto(default)|yes|no
-  freeDiameter: /root/open5gs/install/etc/freeDiameter/smf.conf
+#  freeDiameter: /root/open5gs/install/etc/freeDiameter/smf.conf
+  info:
+    - s_nssai:
+        - sst: 1
+          sd: 000002
+          dnn:
+            - internet
+      tai:
+        - plmn_id:
+            mcc: 001
+            mnc: 01
+          tac: 1
 
 ################################################################################
 # SMF Info
```

<a id="changes_up1"></a>

### Changes in configuration files of Open5GS 5GC U-Plane1

- `open5gs/install/etc/open5gs/upf.yaml`
```diff
--- upf.yaml.orig       2024-03-24 15:36:48.000000000 +0900
+++ upf.yaml    2024-03-25 20:16:20.324142755 +0900
@@ -10,16 +10,17 @@
 upf:
   pfcp:
     server:
-      - address: 127.0.0.7
+      - address: 192.168.0.114
     client:
 #      smf:     #  UPF PFCP Client try to associate SMF PFCP Server
 #        - address: 127.0.0.4
   gtpu:
     server:
-      - address: 127.0.0.7
+      - address: 192.168.0.114
   session:
     - subnet: 10.45.0.1/16
-    - subnet: 2001:db8:cafe::1/48
+      dnn: internet
+      dev: ogstun
   metrics:
     server:
       - address: 127.0.0.7
```

<a id="changes_up2"></a>

### Changes in configuration files of Open5GS 5GC U-Plane2

- `open5gs/install/etc/open5gs/upf.yaml`
```diff
--- upf.yaml.orig       2024-03-24 15:36:48.000000000 +0900
+++ upf.yaml    2024-03-25 20:18:50.813311682 +0900
@@ -10,16 +10,17 @@
 upf:
   pfcp:
     server:
-      - address: 127.0.0.7
+      - address: 192.168.0.115
     client:
 #      smf:     #  UPF PFCP Client try to associate SMF PFCP Server
 #        - address: 127.0.0.4
   gtpu:
     server:
-      - address: 127.0.0.7
+      - address: 192.168.0.115
   session:
-    - subnet: 10.45.0.1/16
-    - subnet: 2001:db8:cafe::1/48
+    - subnet: 10.46.0.1/16
+      dnn: internet
+      dev: ogstun
   metrics:
     server:
       - address: 127.0.0.7
```

<a id="changes_ueransim"></a>

### Changes in configuration files of UERANSIM UE / RAN

<a id="changes_ran"></a>

#### Changes in configuration files of RAN (gNodeB)

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

<a id="changes_ue_sd1"></a>

#### Changes in configuration files of UE[SST:1, SD:0x000001] (IMSI-001010000000000)

- `UERANSIM/config/open5gs-ue-sd1.yaml`
```diff
--- open5gs-ue.yaml.orig        2023-12-02 06:14:20.000000000 +0900
+++ open5gs-ue-sd1.yaml 2024-03-25 20:40:16.465891137 +0900
@@ -1,9 +1,9 @@
 # IMSI number of the UE. IMSI = [MCC|MNC|MSISDN] (In total 15 digits)
-supi: 'imsi-999700000000001'
+supi: 'imsi-001010000000000'
 # Mobile Country Code value of HPLMN
-mcc: '999'
+mcc: '001'
 # Mobile Network Code value of HPLMN (2 or 3 digits)
-mnc: '70'
+mnc: '01'
 # SUCI Protection Scheme : 0 for Null-scheme, 1 for Profile A and 2 for Profile B
 protectionScheme: 0
 # Home Network Public Key for protecting with SUCI Profile A
@@ -28,7 +28,7 @@
 
 # List of gNB IP addresses for Radio Link Simulation
 gnbSearchList:
-  - 127.0.0.1
+  - 192.168.0.131
 
 # UAC Access Identities Configuration
 uacAic:
@@ -50,15 +50,17 @@
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

<a id="changes_ue_sd2"></a>

#### Changes in configuration files of UE[SST:1, SD:0x000002] (IMSI-001010000000000)

- `UERANSIM/config/open5gs-ue-sd2.yaml`
```diff
--- open5gs-ue.yaml.orig        2023-12-02 06:14:20.000000000 +0900
+++ open5gs-ue-sd2.yaml 2024-03-25 20:41:32.181352946 +0900
@@ -1,9 +1,9 @@
 # IMSI number of the UE. IMSI = [MCC|MNC|MSISDN] (In total 15 digits)
-supi: 'imsi-999700000000001'
+supi: 'imsi-001010000000000'
 # Mobile Country Code value of HPLMN
-mcc: '999'
+mcc: '001'
 # Mobile Network Code value of HPLMN (2 or 3 digits)
-mnc: '70'
+mnc: '01'
 # SUCI Protection Scheme : 0 for Null-scheme, 1 for Profile A and 2 for Profile B
 protectionScheme: 0
 # Home Network Public Key for protecting with SUCI Profile A
@@ -28,7 +28,7 @@
 
 # List of gNB IP addresses for Radio Link Simulation
 gnbSearchList:
-  - 127.0.0.1
+  - 192.168.0.131
 
 # UAC Access Identities Configuration
 uacAic:
@@ -50,15 +50,17 @@
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

<a id="network_settings"></a>

## Network settings of Open5GS 5GC and UERANSIM UE / RAN

<a id="network_settings_cp"></a>

### Network settings of Open5GS 5GC C-Plane

Add IP addresses for SMF1 and SMF2.
```
ip addr add 192.168.0.112/24 dev enp0s8
ip addr add 192.168.0.113/24 dev enp0s8
```
**Note. `enp0s8` is the network interface of `192.168.0.0/24` in my VirtualBox environment.
Please change it according to your environment.**

<a id="network_settings_up1"></a>

### Network settings of Open5GS 5GC U-Plane1

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

<a id="network_settings_up2"></a>

### Network settings of Open5GS 5GC U-Plane2

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

<a id="build"></a>

## Build Open5GS and UERANSIM

Please refer to the following for building Open5GS and UERANSIM respectively.
- Open5GS v2.7.0 (2024.03.24) - https://open5gs.org/open5gs/docs/guide/02-building-open5gs-from-sources/
- UERANSIM v3.2.6 (2024.03.08) - https://github.com/aligungr/UERANSIM/wiki/Installation

Install MongoDB on Open5GS 5GC C-Plane machine.
It is not necessary to install MongoDB on Open5GS 5GC U-Plane machines.
[MongoDB Compass](https://www.mongodb.com/products/compass) is a convenient tool to look at the MongoDB database.

<a id="run"></a>

## Run Open5GS 5GC and UERANSIM UE / RAN

First run the 5GC, then UERANSIM (UE & RAN implementation).

<a id="run_cp"></a>

### Run Open5GS 5GC C-Plane

First, run Open5GS 5GC C-Plane.

- Open5GS 5GC C-Plane
```
./install/bin/open5gs-nrfd &
sleep 2
./install/bin/open5gs-scpd &
sleep 2
./install/bin/open5gs-amfd &
sleep 2
./install/bin/open5gs-smfd -c install/etc/open5gs/smf1.yaml &
./install/bin/open5gs-smfd -c install/etc/open5gs/smf2.yaml &
./install/bin/open5gs-ausfd &
./install/bin/open5gs-udmd &
./install/bin/open5gs-udrd &
./install/bin/open5gs-pcfd &
./install/bin/open5gs-nssfd &
./install/bin/open5gs-bsfd &
```

<a id="run_up"></a>

### Run Open5GS 5GC U-Plane1 & U-Plane2

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

<a id="run_ran"></a>

### Run UERANSIM (gNodeB)

Please refer to the following for usage of UERANSIM.

https://github.com/aligungr/UERANSIM/wiki/Usage

```
# ./nr-gnb -c ../config/open5gs-gnb.yaml
UERANSIM v3.2.6
[2024-03-25 21:36:11.999] [sctp] [info] Trying to establish SCTP connection... (192.168.0.111:38412)
[2024-03-25 21:36:12.011] [sctp] [info] SCTP connection established (192.168.0.111:38412)
[2024-03-25 21:36:12.012] [sctp] [debug] SCTP association setup ascId[8]
[2024-03-25 21:36:12.013] [ngap] [debug] Sending NG Setup Request
[2024-03-25 21:36:12.038] [ngap] [debug] NG Setup Response received
[2024-03-25 21:36:12.038] [ngap] [info] NG Setup procedure is successful
```
The Open5GS C-Plane log when executed is as follows.
```
03/25 21:36:12.006: [amf] INFO: gNB-N2 accepted[192.168.0.131]:52087 in ng-path module (../src/amf/ngap-sctp.c:113)
03/25 21:36:12.006: [amf] INFO: gNB-N2 accepted[192.168.0.131] in master_sm module (../src/amf/amf-sm.c:754)
03/25 21:36:12.029: [amf] INFO: [Added] Number of gNBs is now 1 (../src/amf/context.c:1236)
03/25 21:36:12.030: [amf] INFO: gNB-N2[192.168.0.131] max_num_of_ostreams : 10 (../src/amf/amf-sm.c:793)
```

<a id="run_sd1"></a>

### Run UERANSIM (UE[SST:1, SD:0x000001])

Confirm that the packet goes through the DN of U-Plane1 based on SST:1 and SD:0x000001.

<a id="con_sd1"></a>

#### UE connects to U-Plane1 based on SST:1 and SD:0x000001

```
# ./nr-ue -c ../config/open5gs-ue-sd1.yaml 
UERANSIM v3.2.6
[2024-03-25 21:36:48.677] [nas] [info] UE switches to state [MM-DEREGISTERED/PLMN-SEARCH]
[2024-03-25 21:36:48.678] [rrc] [debug] New signal detected for cell[1], total [1] cells in coverage
[2024-03-25 21:36:48.679] [nas] [info] Selected plmn[001/01]
[2024-03-25 21:36:48.679] [rrc] [info] Selected cell plmn[001/01] tac[1] category[SUITABLE]
[2024-03-25 21:36:48.680] [nas] [info] UE switches to state [MM-DEREGISTERED/PS]
[2024-03-25 21:36:48.681] [nas] [info] UE switches to state [MM-DEREGISTERED/NORMAL-SERVICE]
[2024-03-25 21:36:48.681] [nas] [debug] Initial registration required due to [MM-DEREG-NORMAL-SERVICE]
[2024-03-25 21:36:48.684] [nas] [debug] UAC access attempt is allowed for identity[0], category[MO_sig]
[2024-03-25 21:36:48.685] [nas] [debug] Sending Initial Registration
[2024-03-25 21:36:48.685] [rrc] [debug] Sending RRC Setup Request
[2024-03-25 21:36:48.686] [nas] [info] UE switches to state [MM-REGISTER-INITIATED]
[2024-03-25 21:36:48.687] [rrc] [info] RRC connection established
[2024-03-25 21:36:48.687] [rrc] [info] UE switches to state [RRC-CONNECTED]
[2024-03-25 21:36:48.688] [nas] [info] UE switches to state [CM-CONNECTED]
[2024-03-25 21:36:48.714] [nas] [debug] Authentication Request received
[2024-03-25 21:36:48.715] [nas] [debug] Received SQN [000000000181]
[2024-03-25 21:36:48.715] [nas] [debug] SQN-MS [000000000000]
[2024-03-25 21:36:48.731] [nas] [debug] Security Mode Command received
[2024-03-25 21:36:48.732] [nas] [debug] Selected integrity[2] ciphering[0]
[2024-03-25 21:36:48.776] [nas] [debug] Registration accept received
[2024-03-25 21:36:48.777] [nas] [info] UE switches to state [MM-REGISTERED/NORMAL-SERVICE]
[2024-03-25 21:36:48.777] [nas] [debug] Sending Registration Complete
[2024-03-25 21:36:48.778] [nas] [info] Initial Registration is successful
[2024-03-25 21:36:48.778] [nas] [debug] Sending PDU Session Establishment Request
[2024-03-25 21:36:48.778] [nas] [debug] UAC access attempt is allowed for identity[0], category[MO_sig]
[2024-03-25 21:36:48.986] [nas] [debug] Configuration Update Command received
[2024-03-25 21:36:49.041] [nas] [debug] PDU Session Establishment Accept received
[2024-03-25 21:36:49.047] [nas] [info] PDU Session establishment is successful PSI[1]
[2024-03-25 21:36:49.091] [app] [info] Connection setup for PDU session[1] is successful, TUN interface[uesimtun0, 10.45.0.2] is up.
```
The Open5GS C-Plane log when executed is as follows.
```
03/25 21:36:48.679: [amf] INFO: InitialUEMessage (../src/amf/ngap-handler.c:401)
03/25 21:36:48.679: [amf] INFO: [Added] Number of gNB-UEs is now 1 (../src/amf/context.c:2656)
03/25 21:36:48.679: [amf] INFO:     RAN_UE_NGAP_ID[1] AMF_UE_NGAP_ID[1] TAC[1] CellID[0x10] (../src/amf/ngap-handler.c:562)
03/25 21:36:48.680: [amf] INFO: [suci-0-001-01-0000-0-0-0000000000] Unknown UE by SUCI (../src/amf/context.c:1840)
03/25 21:36:48.680: [amf] INFO: [Added] Number of AMF-UEs is now 1 (../src/amf/context.c:1621)
03/25 21:36:48.680: [gmm] INFO: Registration request (../src/amf/gmm-sm.c:1224)
03/25 21:36:48.680: [gmm] INFO: [suci-0-001-01-0000-0-0-0000000000]    SUCI (../src/amf/gmm-handler.c:172)
03/25 21:36:48.687: [sbi] WARNING: [UDM] (NRF-discover) NF has already been added [41918884-eaa4-41ee-ac2d-15ec8e524490:1] (../lib/sbi/nnrf-handler.c:1162)
03/25 21:36:48.687: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.12:80] (../lib/sbi/context.c:2210)
03/25 21:36:48.688: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.12:7777] (../lib/sbi/context.c:1946)
03/25 21:36:48.689: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.12:7777] (../lib/sbi/context.c:1946)
03/25 21:36:48.689: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.12:7777] (../lib/sbi/context.c:1946)
03/25 21:36:48.690: [sbi] INFO: [UDM] (NF-discover) NF Profile updated [41918884-eaa4-41ee-ac2d-15ec8e524490:1] (../lib/sbi/nnrf-handler.c:1200)
03/25 21:36:48.700: [sbi] INFO: [UDM] (SCP-discover) NF registered [41918884-eaa4-41ee-ac2d-15ec8e524490:1] (../lib/sbi/path.c:211)
03/25 21:36:48.756: [sbi] WARNING: [UDR] (NRF-discover) NF has already been added [4199dfa2-eaa4-41ee-92fa-a9d99a609a0f:1] (../lib/sbi/nnrf-handler.c:1162)
03/25 21:36:48.757: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.20:80] (../lib/sbi/context.c:2210)
03/25 21:36:48.758: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.20:7777] (../lib/sbi/context.c:1946)
03/25 21:36:48.758: [sbi] INFO: [UDR] (NF-discover) NF Profile updated [4199dfa2-eaa4-41ee-92fa-a9d99a609a0f:1] (../lib/sbi/nnrf-handler.c:1200)
03/25 21:36:48.761: [sbi] INFO: [UDR] (SCP-discover) NF registered [4199dfa2-eaa4-41ee-92fa-a9d99a609a0f:1] (../lib/sbi/path.c:211)
03/25 21:36:48.973: [gmm] INFO: [imsi-001010000000000] Registration complete (../src/amf/gmm-sm.c:2321)
03/25 21:36:48.973: [amf] INFO: [imsi-001010000000000] Configuration update command (../src/amf/nas-path.c:591)
03/25 21:36:48.974: [gmm] INFO:     UTC [2024-03-25T12:36:48] Timezone[0]/DST[0] (../src/amf/gmm-build.c:558)
03/25 21:36:48.974: [gmm] INFO:     LOCAL [2024-03-25T21:36:48] Timezone[32400]/DST[0] (../src/amf/gmm-build.c:563)
03/25 21:36:48.975: [amf] INFO: [Added] Number of AMF-Sessions is now 1 (../src/amf/context.c:2677)
03/25 21:36:48.976: [gmm] INFO: UE SUPI[imsi-001010000000000] DNN[internet] S_NSSAI[SST:1 SD:0x1] smContextRef [NULL] (../src/amf/gmm-handler.c:1285)
03/25 21:36:48.976: [gmm] INFO: SMF Instance [41ae89f2-eaa4-41ee-b083-05ab4ba5684f] (../src/amf/gmm-handler.c:1324)
03/25 21:36:48.980: [smf] INFO: [Added] Number of SMF-UEs is now 1 (../src/smf/context.c:1019)
03/25 21:36:48.980: [smf] INFO: [Added] Number of SMF-Sessions is now 1 (../src/smf/context.c:3090)
03/25 21:36:48.984: [sbi] WARNING: [UDM] (NRF-discover) NF has already been added [41918884-eaa4-41ee-ac2d-15ec8e524490:1] (../lib/sbi/nnrf-handler.c:1162)
03/25 21:36:48.984: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.12:80] (../lib/sbi/context.c:2210)
03/25 21:36:48.985: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.12:7777] (../lib/sbi/context.c:1946)
03/25 21:36:48.985: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.12:7777] (../lib/sbi/context.c:1946)
03/25 21:36:48.986: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.12:7777] (../lib/sbi/context.c:1946)
03/25 21:36:48.986: [sbi] INFO: [UDM] (NF-discover) NF Profile updated [41918884-eaa4-41ee-ac2d-15ec8e524490:1] (../lib/sbi/nnrf-handler.c:1200)
03/25 21:36:48.992: [sbi] INFO: [UDM] (SCP-discover) NF registered [41918884-eaa4-41ee-ac2d-15ec8e524490:1] (../lib/sbi/path.c:211)
03/25 21:36:48.996: [sbi] WARNING: [PCF] (NRF-discover) NF has already been added [419a69cc-eaa4-41ee-8a01-d10531f1e6cd:1] (../lib/sbi/nnrf-handler.c:1162)
03/25 21:36:48.997: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.13:80] (../lib/sbi/context.c:2210)
03/25 21:36:48.998: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.13:7777] (../lib/sbi/context.c:1946)
03/25 21:36:48.998: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.13:7777] (../lib/sbi/context.c:1946)
03/25 21:36:48.998: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.13:7777] (../lib/sbi/context.c:1946)
03/25 21:36:48.999: [sbi] INFO: [PCF] (NF-discover) NF Profile updated [419a69cc-eaa4-41ee-8a01-d10531f1e6cd:1] (../lib/sbi/nnrf-handler.c:1200)
03/25 21:36:49.007: [sbi] WARNING: [UDR] (NRF-discover) NF has already been added [4199dfa2-eaa4-41ee-92fa-a9d99a609a0f:1] (../lib/sbi/nnrf-handler.c:1162)
03/25 21:36:49.008: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.20:80] (../lib/sbi/context.c:2210)
03/25 21:36:49.008: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.20:7777] (../lib/sbi/context.c:1946)
03/25 21:36:49.008: [sbi] INFO: [UDR] (NF-discover) NF Profile updated [4199dfa2-eaa4-41ee-92fa-a9d99a609a0f:1] (../lib/sbi/nnrf-handler.c:1200)
03/25 21:36:49.012: [sbi] WARNING: [UDR] (SCP-discover) NF has already been added [4199dfa2-eaa4-41ee-92fa-a9d99a609a0f:2] (../lib/sbi/path.c:216)
03/25 21:36:49.014: [sbi] WARNING: [BSF] (NRF-discover) NF has already been added [4190b508-eaa4-41ee-84c2-851c47f6e952:1] (../lib/sbi/nnrf-handler.c:1162)
03/25 21:36:49.015: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.15:80] (../lib/sbi/context.c:2210)
03/25 21:36:49.015: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.15:7777] (../lib/sbi/context.c:1946)
03/25 21:36:49.016: [sbi] INFO: [BSF] (NF-discover) NF Profile updated [4190b508-eaa4-41ee-84c2-851c47f6e952:1] (../lib/sbi/nnrf-handler.c:1200)
03/25 21:36:49.019: [sbi] INFO: [BSF] (SCP-discover) NF registered [4190b508-eaa4-41ee-84c2-851c47f6e952:1] (../lib/sbi/path.c:211)
03/25 21:36:49.022: [sbi] INFO: [PCF] (SCP-discover) NF registered [419a69cc-eaa4-41ee-8a01-d10531f1e6cd:1] (../lib/sbi/path.c:211)
03/25 21:36:49.023: [smf] INFO: UE SUPI[imsi-001010000000000] DNN[internet] IPv4[10.45.0.2] IPv6[] (../src/smf/npcf-handler.c:542)
03/25 21:36:49.025: [gtp] INFO: gtp_connect() [192.168.0.114]:2152 (../lib/gtp/path.c:60)
03/25 21:36:49.035: [sbi] WARNING: [UDM] (NRF-discover) NF has already been added [41918884-eaa4-41ee-ac2d-15ec8e524490:1] (../lib/sbi/nnrf-handler.c:1162)
03/25 21:36:49.036: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.12:80] (../lib/sbi/context.c:2210)
03/25 21:36:49.036: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.12:7777] (../lib/sbi/context.c:1946)
03/25 21:36:49.037: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.12:7777] (../lib/sbi/context.c:1946)
03/25 21:36:49.037: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.12:7777] (../lib/sbi/context.c:1946)
03/25 21:36:49.037: [sbi] INFO: [UDM] (NF-discover) NF Profile updated [41918884-eaa4-41ee-ac2d-15ec8e524490:1] (../lib/sbi/nnrf-handler.c:1200)
03/25 21:36:49.041: [sbi] WARNING: [UDM] (SCP-discover) NF has already been added [41918884-eaa4-41ee-ac2d-15ec8e524490:2] (../lib/sbi/path.c:216)
03/25 21:36:49.042: [amf] INFO: [imsi-001010000000000:1:11][0:0:NULL] /nsmf-pdusession/v1/sm-contexts/{smContextRef}/modify (../src/amf/nsmf-handler.c:867)
```
The Open5GS U-Plane1 log when executed is as follows.
```
03/25 21:36:48.947: [upf] INFO: [Added] Number of UPF-Sessions is now 1 (../src/upf/context.c:208)
03/25 21:36:48.947: [gtp] INFO: gtp_connect() [192.168.0.112]:2152 (../lib/gtp/path.c:60)
03/25 21:36:48.947: [upf] INFO: UE F-SEID[UP:0x84f CP:0x8eb] APN[internet] PDN-Type[1] IPv4[10.45.0.2] IPv6[] (../src/upf/context.c:485)
03/25 21:36:48.947: [upf] INFO: UE F-SEID[UP:0x84f CP:0x8eb] APN[internet] PDN-Type[1] IPv4[10.45.0.2] IPv6[] (../src/upf/context.c:485)
03/25 21:36:48.956: [gtp] INFO: gtp_connect() [192.168.0.131]:2152 (../lib/gtp/path.c:60)
```
The TUNnel interface `uesimtun0` is created as follows.
```
# ip addr show
...
9: uesimtun0: <POINTOPOINT,PROMISC,NOTRAILERS,UP,LOWER_UP> mtu 1400 qdisc fq_codel state UNKNOWN group default qlen 500
    link/none 
    inet 10.45.0.2/32 scope global uesimtun0
       valid_lft forever preferred_lft forever
    inet6 fe80::2dff:f15d:c28f:b51c/64 scope link stable-privacy 
       valid_lft forever preferred_lft forever
...
```

<a id="ping_sd1"></a>

#### Ping google.com going through DN=10.45.0.0/16 on U-Plane1

Confirm by using `tcpdump` that the packet goes through `if=ogstun` on U-Plane1.
```
# ping google.com -I uesimtun0 -n
PING google.com (142.250.198.14) from 10.45.0.2 uesimtun0: 56(84) bytes of data.
64 bytes from 142.250.198.14: icmp_seq=1 ttl=61 time=31.9 ms
64 bytes from 142.250.198.14: icmp_seq=2 ttl=61 time=16.5 ms
64 bytes from 142.250.198.14: icmp_seq=3 ttl=61 time=21.3 ms
```
The `tcpdump` log on U-Plane1 is as follows.
```
21:38:35.545703 IP 10.45.0.2 > 142.250.198.14: ICMP echo request, id 6, seq 1, length 64
21:38:35.574278 IP 142.250.198.14 > 10.45.0.2: ICMP echo reply, id 6, seq 1, length 64
21:38:36.545310 IP 10.45.0.2 > 142.250.198.14: ICMP echo request, id 6, seq 2, length 64
21:38:36.561089 IP 142.250.198.14 > 10.45.0.2: ICMP echo reply, id 6, seq 2, length 64
21:38:37.547232 IP 10.45.0.2 > 142.250.198.14: ICMP echo request, id 6, seq 3, length 64
21:38:37.566185 IP 142.250.198.14 > 10.45.0.2: ICMP echo reply, id 6, seq 3, length 64
```
**Note. Make sure the packet does not go through U-Plane2.**

<a id="run_sd2"></a>

### Run UERANSIM (UE[SST:1, SD:0x000002])

Then the UE disconnects from gNodeB and connects to gNodeB using the configuration file for SST:1 and SD:0x000002.
Confirm that the packet goes through the DN of U-Plane2 based on SST:1 and SD:0x000002.

<a id="con_sd2"></a>

#### UE connects to U-Plane2 based on SST:1 and SD:0x000002

```
# ./nr-ue -c ../config/open5gs-ue-sd2.yaml 
UERANSIM v3.2.6
[2024-03-25 21:39:49.839] [nas] [info] UE switches to state [MM-DEREGISTERED/PLMN-SEARCH]
[2024-03-25 21:39:49.841] [rrc] [debug] New signal detected for cell[1], total [1] cells in coverage
[2024-03-25 21:39:52.024] [nas] [info] Selected plmn[001/01]
[2024-03-25 21:39:52.024] [rrc] [info] Selected cell plmn[001/01] tac[1] category[SUITABLE]
[2024-03-25 21:39:52.025] [nas] [info] UE switches to state [MM-DEREGISTERED/PS]
[2024-03-25 21:39:52.026] [nas] [info] UE switches to state [MM-DEREGISTERED/NORMAL-SERVICE]
[2024-03-25 21:39:52.026] [nas] [debug] Initial registration required due to [MM-DEREG-NORMAL-SERVICE]
[2024-03-25 21:39:52.029] [nas] [debug] UAC access attempt is allowed for identity[0], category[MO_sig]
[2024-03-25 21:39:52.030] [nas] [debug] Sending Initial Registration
[2024-03-25 21:39:52.031] [rrc] [debug] Sending RRC Setup Request
[2024-03-25 21:39:52.031] [nas] [info] UE switches to state [MM-REGISTER-INITIATED]
[2024-03-25 21:39:52.033] [rrc] [info] RRC connection established
[2024-03-25 21:39:52.033] [rrc] [info] UE switches to state [RRC-CONNECTED]
[2024-03-25 21:39:52.034] [nas] [info] UE switches to state [CM-CONNECTED]
[2024-03-25 21:39:52.073] [nas] [debug] Authentication Request received
[2024-03-25 21:39:52.074] [nas] [debug] Received SQN [0000000001A1]
[2024-03-25 21:39:52.075] [nas] [debug] SQN-MS [000000000000]
[2024-03-25 21:39:52.090] [nas] [debug] Security Mode Command received
[2024-03-25 21:39:52.091] [nas] [debug] Selected integrity[2] ciphering[0]
[2024-03-25 21:39:52.125] [nas] [debug] Registration accept received
[2024-03-25 21:39:52.126] [nas] [info] UE switches to state [MM-REGISTERED/NORMAL-SERVICE]
[2024-03-25 21:39:52.126] [nas] [debug] Sending Registration Complete
[2024-03-25 21:39:52.127] [nas] [info] Initial Registration is successful
[2024-03-25 21:39:52.127] [nas] [debug] Sending PDU Session Establishment Request
[2024-03-25 21:39:52.128] [nas] [debug] UAC access attempt is allowed for identity[0], category[MO_sig]
[2024-03-25 21:39:52.331] [nas] [debug] Configuration Update Command received
[2024-03-25 21:39:52.367] [nas] [debug] PDU Session Establishment Accept received
[2024-03-25 21:39:52.372] [nas] [info] PDU Session establishment is successful PSI[1]
[2024-03-25 21:39:52.394] [app] [info] Connection setup for PDU session[1] is successful, TUN interface[uesimtun0, 10.46.0.2] is up.
```
The Open5GS C-Plane log when executed is as follows.
```
03/25 21:39:52.025: [amf] INFO: InitialUEMessage (../src/amf/ngap-handler.c:401)
03/25 21:39:52.025: [amf] INFO: [Added] Number of gNB-UEs is now 2 (../src/amf/context.c:2656)
03/25 21:39:52.025: [amf] INFO:     RAN_UE_NGAP_ID[2] AMF_UE_NGAP_ID[2] TAC[1] CellID[0x10] (../src/amf/ngap-handler.c:562)
03/25 21:39:52.025: [amf] INFO: [suci-0-001-01-0000-0-0-0000000000] known UE by SUCI (../src/amf/context.c:1838)
03/25 21:39:52.025: [amf] WARNING: [suci-0-001-01-0000-0-0-0000000000] Holding NG Context (../src/amf/amf-sm.c:965)
03/25 21:39:52.025: [amf] WARNING: [suci-0-001-01-0000-0-0-0000000000]    RAN_UE_NGAP_ID[1] AMF_UE_NGAP_ID[1] (../src/amf/amf-sm.c:965)
03/25 21:39:52.025: [gmm] INFO: Registration request (../src/amf/gmm-sm.c:1224)
03/25 21:39:52.025: [gmm] INFO: [suci-0-001-01-0000-0-0-0000000000]    SUCI (../src/amf/gmm-handler.c:172)
03/25 21:39:52.045: [smf] INFO: Removed Session: UE IMSI:[imsi-001010000000000] DNN:[internet:1] IPv4:[10.45.0.2] IPv6:[] (../src/smf/context.c:1677)
03/25 21:39:52.045: [smf] INFO: [Removed] Number of SMF-Sessions is now 0 (../src/smf/context.c:3098)
03/25 21:39:52.046: [smf] INFO: [Removed] Number of SMF-UEs is now 0 (../src/smf/context.c:1080)
03/25 21:39:52.048: [amf] INFO: [imsi-001010000000000:1] Release SM context [204] (../src/amf/amf-sm.c:505)
03/25 21:39:52.048: [amf] INFO: [imsi-001010000000000:1] Release SM Context [state:31] (../src/amf/nsmf-handler.c:1082)
03/25 21:39:52.049: [amf] INFO: [Removed] Number of AMF-Sessions is now 0 (../src/amf/context.c:2684)
03/25 21:39:52.083: [gmm] WARNING: [suci-0-001-01-0000-0-0-0000000000] Clear NG Context (../src/amf/gmm-sm.c:2002)
03/25 21:39:52.084: [gmm] WARNING: [suci-0-001-01-0000-0-0-0000000000]    RAN_UE_NGAP_ID[1] AMF_UE_NGAP_ID[1] (../src/amf/gmm-sm.c:2002)
03/25 21:39:52.087: [amf] INFO: UE Context Release [Action:1] (../src/amf/ngap-handler.c:1696)
03/25 21:39:52.087: [amf] INFO:     RAN_UE_NGAP_ID[1] AMF_UE_NGAP_ID[1] (../src/amf/ngap-handler.c:1697)
03/25 21:39:52.088: [amf] INFO: [Removed] Number of gNB-UEs is now 1 (../src/amf/context.c:2663)
03/25 21:39:52.108: [pcf] WARNING: NF EndPoint(addr) updated [127.0.0.5:7777] (../src/pcf/npcf-handler.c:113)
03/25 21:39:52.317: [gmm] INFO: [imsi-001010000000000] Registration complete (../src/amf/gmm-sm.c:2321)
03/25 21:39:52.318: [amf] INFO: [imsi-001010000000000] Configuration update command (../src/amf/nas-path.c:591)
03/25 21:39:52.318: [gmm] INFO:     UTC [2024-03-25T12:39:52] Timezone[0]/DST[0] (../src/amf/gmm-build.c:558)
03/25 21:39:52.319: [gmm] INFO:     LOCAL [2024-03-25T21:39:52] Timezone[32400]/DST[0] (../src/amf/gmm-build.c:563)
03/25 21:39:52.320: [amf] INFO: [Added] Number of AMF-Sessions is now 1 (../src/amf/context.c:2677)
03/25 21:39:52.321: [gmm] INFO: UE SUPI[imsi-001010000000000] DNN[internet] S_NSSAI[SST:1 SD:0x2] smContextRef [NULL] (../src/amf/gmm-handler.c:1285)
03/25 21:39:52.322: [gmm] INFO: SMF Instance [41add958-eaa4-41ee-910e-31d3641cb739] (../src/amf/gmm-handler.c:1324)
03/25 21:39:52.326: [smf] INFO: [Added] Number of SMF-UEs is now 1 (../src/smf/context.c:1019)
03/25 21:39:52.327: [smf] INFO: [Added] Number of SMF-Sessions is now 1 (../src/smf/context.c:3090)
03/25 21:39:52.331: [sbi] WARNING: [UDM] (NRF-discover) NF has already been added [41918884-eaa4-41ee-ac2d-15ec8e524490:1] (../lib/sbi/nnrf-handler.c:1162)
03/25 21:39:52.332: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.12:80] (../lib/sbi/context.c:2210)
03/25 21:39:52.332: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.12:7777] (../lib/sbi/context.c:1946)
03/25 21:39:52.333: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.12:7777] (../lib/sbi/context.c:1946)
03/25 21:39:52.333: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.12:7777] (../lib/sbi/context.c:1946)
03/25 21:39:52.334: [sbi] INFO: [UDM] (NF-discover) NF Profile updated [41918884-eaa4-41ee-ac2d-15ec8e524490:1] (../lib/sbi/nnrf-handler.c:1200)
03/25 21:39:52.342: [sbi] INFO: [UDM] (SCP-discover) NF registered [41918884-eaa4-41ee-ac2d-15ec8e524490:1] (../lib/sbi/path.c:211)
03/25 21:39:52.346: [sbi] WARNING: [PCF] (NRF-discover) NF has already been added [419a69cc-eaa4-41ee-8a01-d10531f1e6cd:1] (../lib/sbi/nnrf-handler.c:1162)
03/25 21:39:52.346: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.13:80] (../lib/sbi/context.c:2210)
03/25 21:39:52.346: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.13:7777] (../lib/sbi/context.c:1946)
03/25 21:39:52.346: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.13:7777] (../lib/sbi/context.c:1946)
03/25 21:39:52.347: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.13:7777] (../lib/sbi/context.c:1946)
03/25 21:39:52.347: [sbi] INFO: [PCF] (NF-discover) NF Profile updated [419a69cc-eaa4-41ee-8a01-d10531f1e6cd:1] (../lib/sbi/nnrf-handler.c:1200)
03/25 21:39:52.348: [sbi] WARNING: [UDR] (NRF-discover) NF has already been added [4199dfa2-eaa4-41ee-92fa-a9d99a609a0f:1] (../lib/sbi/nnrf-handler.c:1162)
03/25 21:39:52.348: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.20:80] (../lib/sbi/context.c:2210)
03/25 21:39:52.349: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.20:7777] (../lib/sbi/context.c:1946)
03/25 21:39:52.349: [sbi] INFO: [UDR] (NF-discover) NF Profile updated [4199dfa2-eaa4-41ee-92fa-a9d99a609a0f:1] (../lib/sbi/nnrf-handler.c:1200)
03/25 21:39:52.350: [sbi] WARNING: [UDR] (SCP-discover) NF has already been added [4199dfa2-eaa4-41ee-92fa-a9d99a609a0f:2] (../lib/sbi/path.c:216)
03/25 21:39:52.351: [sbi] WARNING: [BSF] (NRF-discover) NF has already been added [4190b508-eaa4-41ee-84c2-851c47f6e952:1] (../lib/sbi/nnrf-handler.c:1162)
03/25 21:39:52.351: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.15:80] (../lib/sbi/context.c:2210)
03/25 21:39:52.351: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.15:7777] (../lib/sbi/context.c:1946)
03/25 21:39:52.351: [sbi] INFO: [BSF] (NF-discover) NF Profile updated [4190b508-eaa4-41ee-84c2-851c47f6e952:1] (../lib/sbi/nnrf-handler.c:1200)
03/25 21:39:52.352: [sbi] WARNING: [BSF] (SCP-discover) NF has already been added [4190b508-eaa4-41ee-84c2-851c47f6e952:1] (../lib/sbi/path.c:216)
03/25 21:39:52.353: [sbi] INFO: [PCF] (SCP-discover) NF registered [419a69cc-eaa4-41ee-8a01-d10531f1e6cd:1] (../lib/sbi/path.c:211)
03/25 21:39:52.353: [smf] INFO: UE SUPI[imsi-001010000000000] DNN[internet] IPv4[10.46.0.2] IPv6[] (../src/smf/npcf-handler.c:542)
03/25 21:39:52.354: [gtp] INFO: gtp_connect() [192.168.0.115]:2152 (../lib/gtp/path.c:60)
03/25 21:39:52.359: [sbi] WARNING: [UDM] (NRF-discover) NF has already been added [41918884-eaa4-41ee-ac2d-15ec8e524490:1] (../lib/sbi/nnrf-handler.c:1162)
03/25 21:39:52.359: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.12:80] (../lib/sbi/context.c:2210)
03/25 21:39:52.360: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.12:7777] (../lib/sbi/context.c:1946)
03/25 21:39:52.360: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.12:7777] (../lib/sbi/context.c:1946)
03/25 21:39:52.360: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.12:7777] (../lib/sbi/context.c:1946)
03/25 21:39:52.360: [sbi] INFO: [UDM] (NF-discover) NF Profile updated [41918884-eaa4-41ee-ac2d-15ec8e524490:1] (../lib/sbi/nnrf-handler.c:1200)
03/25 21:39:52.362: [sbi] WARNING: [UDM] (SCP-discover) NF has already been added [41918884-eaa4-41ee-ac2d-15ec8e524490:2] (../lib/sbi/path.c:216)
03/25 21:39:52.362: [amf] INFO: [imsi-001010000000000:1:11][0:0:NULL] /nsmf-pdusession/v1/sm-contexts/{smContextRef}/modify (../src/amf/nsmf-handler.c:867)
```
The Open5GS U-Plane2 log when executed is as follows.
```
03/25 21:39:52.339: [upf] INFO: [Added] Number of UPF-Sessions is now 1 (../src/upf/context.c:208)
03/25 21:39:52.339: [gtp] INFO: gtp_connect() [192.168.0.113]:2152 (../lib/gtp/path.c:60)
03/25 21:39:52.339: [upf] INFO: UE F-SEID[UP:0x423 CP:0x2af] APN[internet] PDN-Type[1] IPv4[10.46.0.2] IPv6[] (../src/upf/context.c:485)
03/25 21:39:52.339: [upf] INFO: UE F-SEID[UP:0x423 CP:0x2af] APN[internet] PDN-Type[1] IPv4[10.46.0.2] IPv6[] (../src/upf/context.c:485)
03/25 21:39:52.343: [gtp] INFO: gtp_connect() [192.168.0.131]:2152 (../lib/gtp/path.c:60)
```
The TUNnel interface `uesimtun0` is created as follows.
```
# ip addr show
...
10: uesimtun0: <POINTOPOINT,PROMISC,NOTRAILERS,UP,LOWER_UP> mtu 1400 qdisc fq_codel state UNKNOWN group default qlen 500
    link/none 
    inet 10.46.0.2/32 scope global uesimtun0
       valid_lft forever preferred_lft forever
    inet6 fe80::16be:3525:76dc:4db3/64 scope link stable-privacy 
       valid_lft forever preferred_lft forever
...
```

<a id="ping_sd2"></a>

#### Ping google.com going through DN=10.46.0.0/16 on U-Plane2

Confirm by using `tcpdump` that the packet goes through `if=ogstun` on U-Plane2.
```
# ping google.com -I uesimtun0 -n
PING google.com (142.250.198.14) from 10.46.0.2 uesimtun0: 56(84) bytes of data.
64 bytes from 142.250.198.14: icmp_seq=1 ttl=61 time=70.1 ms
64 bytes from 142.250.198.14: icmp_seq=2 ttl=61 time=38.7 ms
64 bytes from 142.250.198.14: icmp_seq=3 ttl=61 time=42.2 ms
```
The `tcpdump` log on U-Plane2 is as follows.
```
21:42:00.636794 IP 10.46.0.2 > 142.250.198.14: ICMP echo request, id 7, seq 1, length 64
21:42:00.704277 IP 142.250.198.14 > 10.46.0.2: ICMP echo reply, id 7, seq 1, length 64
21:42:01.638330 IP 10.46.0.2 > 142.250.198.14: ICMP echo request, id 7, seq 2, length 64
21:42:01.674356 IP 142.250.198.14 > 10.46.0.2: ICMP echo reply, id 7, seq 2, length 64
21:42:02.638285 IP 10.46.0.2 > 142.250.198.14: ICMP echo request, id 7, seq 3, length 64
21:42:02.679587 IP 142.250.198.14 > 10.46.0.2: ICMP echo reply, id 7, seq 3, length 64
```
**Note. Make sure the packet does not go through U-Plane1.**

---
I was able to confirm the very simple configuration in which one UE connects to the UPF based on S-NSSAI. I would like to thank the excellent developers and all the contributors of Open5GS and UERANSIM.

<a id="changelog"></a>

## Changelog (summary)

- [2024.03.25] Updated to Open5GS v2.7.0 (2024.03.24).
- [2023.03.18] Updated to Open5GS v2.6.1 (2023.03.18) and UERANSIM v3.2.6 (2023.03.17).
- [2023.01.13] Updated to Open5GS v2.5.6.
- [2022.08.01] Initial release.
