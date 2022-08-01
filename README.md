# Open5GS 5GC & UERANSIM UE / RAN Sample Configuration - Select UPF based on S-NSSAI
This describes a very simple configuration that uses Open5GS and UERANSIM to select the UPF based on S-NSSAI.

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
  - [Run UERANSIM (UE set to SST:1 and SD:0x000001)](#run_sd1)
    - [Start UE connected to U-Plane1 based on SST:1 and SD:0x000001](#con_sd1)
    - [Ping google.com going through DN=10.45.0.0/16 on U-Plane1](#ping_sd1)
  - [Run UERANSIM (UE set to SST:1 and SD:0x000002)](#run_sd2)
    - [Start UE connected to U-Plane2 based on SST:1 and SD:0x000002](#con_sd2)
    - [Ping google.com going through DN=10.46.0.0/16 on U-Plane2](#ping_sd2)
- [Changelog (summary)](#changelog)

---
<h2 id="overview">Overview of Open5GS 5GC Simulation Mobile Network</h2>

The following minimum configuration was set as a condition.
- The UE selects a pair of SMF and UPF based on S-NSSAI.

The built simulation environment is as follows.

<img src="./images/network-overview.png" title="./images/network-overview.png" width=1000px></img>

The 5GC / UE / RAN used are as follows.
- 5GC - Open5GS v2.4.9 - https://github.com/open5gs/open5gs
- UE / RAN - UERANSIM v3.2.6 - https://github.com/aligungr/UERANSIM

Each VMs are as follows.  
| VM # | SW & Role | IP address | OS | Memory (Min) | HDD (Min) |
| --- | --- | --- | --- | --- | --- |
| VM1 | Open5GS 5GC C-Plane | 192.168.0.111/24 <br> 192.168.0.112/24 <br> 192.168.0.113/24 | Ubuntu 20.04 | 1GB | 20GB |
| VM2 | Open5GS 5GC U-Plane1  | 192.168.0.114/24 | Ubuntu 20.04 | 1GB | 20GB |
| VM3 | Open5GS 5GC U-Plane2  | 192.168.0.115/24 | Ubuntu 20.04 | 1GB | 20GB |
| VM4 | UERANSIM RAN (gNodeB) | 192.168.0.131/24 | Ubuntu 20.04 | 1GB | 10GB |
| VM5 | UERANSIM UE | 192.168.0.132/24 | Ubuntu 20.04 | 1GB | 10GB |

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

<h2 id="changes">Changes in configuration files of Open5GS 5GC and UERANSIM UE / RAN</h2>

Please refer to the following for building Open5GS and UERANSIM respectively.
- Open5GS v2.4.9 - https://open5gs.org/open5gs/docs/guide/02-building-open5gs-from-sources/
- UERANSIM v3.2.6 - https://github.com/aligungr/UERANSIM/wiki/Installation

<h3 id="changes_cp">Changes in configuration files of Open5GS 5GC C-Plane</h3>

- `open5gs/install/etc/open5gs/amf.yaml`
```diff
--- amf.yaml.orig       2022-07-30 23:08:52.000000000 +0900
+++ amf.yaml    2022-07-30 23:12:28.000000000 +0900
@@ -229,25 +229,28 @@
       - addr: 127.0.0.5
         port: 7777
     ngap:
-      - addr: 127.0.0.5
+      - addr: 192.168.0.111
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
--- smf.yaml.orig       2022-07-30 23:08:52.000000000 +0900
+++ smf1.yaml   2022-07-30 23:14:08.000000000 +0900
@@ -381,17 +381,14 @@
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
     subnet:
       - addr: 10.45.0.1/16
-      - addr: 2001:db8:cafe::1/48
+        dnn: internet
     dns:
       - 8.8.8.8
       - 8.8.4.4
@@ -400,7 +397,18 @@
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
 # nrf:
@@ -501,7 +509,8 @@
 #
 upf:
     pfcp:
-      - addr: 127.0.0.7
+      - addr: 192.168.0.114
+        dnn: internet
 
 #
 # parameter:
```
- `open5gs/install/etc/open5gs/smf2.yaml`
```diff
--- smf.yaml.orig       2022-07-30 23:08:52.000000000 +0900
+++ smf2.yaml   2022-07-30 23:15:32.000000000 +0900
@@ -378,20 +378,17 @@
 
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
     subnet:
-      - addr: 10.45.0.1/16
-      - addr: 2001:db8:cafe::1/48
+      - addr: 10.46.0.1/16
+        dnn: internet
     dns:
       - 8.8.8.8
       - 8.8.4.4
@@ -400,7 +397,18 @@
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
 # nrf:
@@ -501,7 +509,8 @@
 #
 upf:
     pfcp:
-      - addr: 127.0.0.7
+      - addr: 192.168.0.115
+        dnn: internet
 
 #
 # parameter:
```
- `open5gs/install/etc/open5gs/nssf.yaml`
```diff
--- nssf.yaml.orig      2022-07-30 23:08:52.000000000 +0900
+++ nssf.yaml   2022-07-31 14:58:14.000000000 +0900
@@ -139,10 +139,16 @@
       - addr: 127.0.0.14
         port: 7777
     nsi:
-      - addr: ::1
+      - addr: 127.0.0.10
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
 # nrf:
```
- `open5gs/install/etc/freeDiameter/smf1.conf`  
`smf1.conf` is equal to the original `smf.conf`.

- `open5gs/install/etc/freeDiameter/smf2.conf`
```diff
--- smf.conf.orig       2021-08-09 14:06:48.000000000 +0000
+++ smf2.conf   2021-08-09 16:01:40.000000000 +0000
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
--- upf.yaml.orig       2022-07-30 23:10:12.000000000 +0900
+++ upf.yaml    2022-07-30 23:14:48.000000000 +0900
@@ -166,12 +166,13 @@
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
 
 #
 # smf:
```

<h3 id="changes_up2">Changes in configuration files of Open5GS 5GC U-Plane2</h3>

- `open5gs/install/etc/open5gs/upf.yaml`
```diff
--- upf.yaml.orig       2022-07-30 23:10:36.000000000 +0900
+++ upf.yaml    2022-07-30 23:15:06.000000000 +0900
@@ -166,12 +166,13 @@
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
 
 #
 # smf:
```

<h3 id="changes_ueransim">Changes in configuration files of UERANSIM UE / RAN</h3>

<h4 id="changes_ran">Changes in configuration files of RAN (gNodeB)</h4>

- `UERANSIM/config/open5gs-gnb.yaml`
```diff
--- open5gs-gnb.yaml.orig       2022-07-03 13:06:44.000000000 +0900
+++ open5gs-gnb.yaml    2022-07-29 20:40:56.000000000 +0900
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
--- open5gs-ue.yaml.orig        2022-07-03 13:06:44.000000000 +0900
+++ open5gs-ue-sd1.yaml 2022-07-29 20:42:40.000000000 +0900
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
 
 # Permanent subscription key
 key: '465B5CE8B199B49FAA5F0A2EE238A6BC'
@@ -20,7 +20,7 @@
 
 # List of gNB IP addresses for Radio Link Simulation
 gnbSearchList:
-  - 127.0.0.1
+  - 192.168.0.131
 
 # UAC Access Identities Configuration
 uacAic:
@@ -42,15 +42,17 @@
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
--- open5gs-ue.yaml.orig        2022-07-03 13:06:44.000000000 +0900
+++ open5gs-ue-sd2.yaml 2022-07-29 20:42:50.000000000 +0900
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
 
 # Permanent subscription key
 key: '465B5CE8B199B49FAA5F0A2EE238A6BC'
@@ -20,7 +20,7 @@
 
 # List of gNB IP addresses for Radio Link Simulation
 gnbSearchList:
-  - 127.0.0.1
+  - 192.168.0.131
 
 # UAC Access Identities Configuration
 uacAic:
@@ -42,15 +42,17 @@
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
- Open5GS v2.4.9 - https://open5gs.org/open5gs/docs/guide/02-building-open5gs-from-sources/
- UERANSIM v3.2.6 - https://github.com/aligungr/UERANSIM/wiki/Installation

Note. Install MongoDB with package manager on Open5GS 5GC C-Plane machine.
[MongoDB Compass](https://www.mongodb.com/products/compass) is a convenient tool to look at the MongoDB database.
```
# apt update
# apt install mongodb
# systemctl start mongodb
# systemctl enable mongodb
```
It is not necessary to install MongoDB on Open5GS 5GC U-Plane machines.

<h2 id="run">Run Open5GS 5GC and UERANSIM UE / RAN</h2>

First run the 5GC, then UERANSIM (UE & RAN implementation).

<h3 id="run_cp">Run Open5GS 5GC C-Plane</h3>

First, run Open5GS 5GC C-Plane.

- Open5GS 5GC C-Plane
```
./install/bin/open5gs-nrfd &
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
[2022-08-01 20:15:30.134] [sctp] [info] Trying to establish SCTP connection... (192.168.0.111:38412)
[2022-08-01 20:15:30.137] [sctp] [info] SCTP connection established (192.168.0.111:38412)
[2022-08-01 20:15:30.137] [sctp] [debug] SCTP association setup ascId[7]
[2022-08-01 20:15:30.137] [ngap] [debug] Sending NG Setup Request
[2022-08-01 20:15:30.138] [ngap] [debug] NG Setup Response received
[2022-08-01 20:15:30.138] [ngap] [info] NG Setup procedure is successful
```
The Open5GS C-Plane log when executed is as follows.
```
08/01 20:15:30.118: [amf] INFO: gNB-N2 accepted[192.168.0.131]:47393 in ng-path module (../src/amf/ngap-sctp.c:113)
08/01 20:15:30.118: [amf] INFO: gNB-N2 accepted[192.168.0.131] in master_sm module (../src/amf/amf-sm.c:660)
08/01 20:15:30.118: [amf] INFO: [Added] Number of gNBs is now 1 (../src/amf/context.c:877)
08/01 20:15:30.118: [amf] INFO: gNB-N2[192.168.0.131] max_num_of_ostreams : 10 (../src/amf/amf-sm.c:699)
```

<h3 id="run_sd1">Run UERANSIM (UE set to SST:1 and SD:0x000001)</h3>

Confirm that the packet goes through the DN of U-Plane1 based on SST:1 and SD:0x000001.

<h4 id="con_sd1">Start UE connected to U-Plane1 based on SST:1 and SD:0x000001</h4>

```
# ./nr-ue -c ../config/open5gs-ue-sd1.yaml 
UERANSIM v3.2.6
[2022-08-01 20:16:02.217] [nas] [info] UE switches to state [MM-DEREGISTERED/PLMN-SEARCH]
[2022-08-01 20:16:02.218] [rrc] [debug] New signal detected for cell[1], total [1] cells in coverage
[2022-08-01 20:16:02.219] [nas] [info] Selected plmn[001/01]
[2022-08-01 20:16:02.219] [rrc] [info] Selected cell plmn[001/01] tac[1] category[SUITABLE]
[2022-08-01 20:16:02.220] [nas] [info] UE switches to state [MM-DEREGISTERED/PS]
[2022-08-01 20:16:02.220] [nas] [info] UE switches to state [MM-DEREGISTERED/NORMAL-SERVICE]
[2022-08-01 20:16:02.221] [nas] [debug] Initial registration required due to [MM-DEREG-NORMAL-SERVICE]
[2022-08-01 20:16:02.221] [nas] [debug] UAC access attempt is allowed for identity[0], category[MO_sig]
[2022-08-01 20:16:02.222] [nas] [debug] Sending Initial Registration
[2022-08-01 20:16:02.222] [rrc] [debug] Sending RRC Setup Request
[2022-08-01 20:16:02.222] [nas] [info] UE switches to state [MM-REGISTER-INITIATED]
[2022-08-01 20:16:02.223] [rrc] [info] RRC connection established
[2022-08-01 20:16:02.223] [rrc] [info] UE switches to state [RRC-CONNECTED]
[2022-08-01 20:16:02.223] [nas] [info] UE switches to state [CM-CONNECTED]
[2022-08-01 20:16:02.230] [nas] [debug] Authentication Request received
[2022-08-01 20:16:02.233] [nas] [debug] Security Mode Command received
[2022-08-01 20:16:02.234] [nas] [debug] Selected integrity[2] ciphering[0]
[2022-08-01 20:16:02.250] [nas] [debug] Registration accept received
[2022-08-01 20:16:02.250] [nas] [info] UE switches to state [MM-REGISTERED/NORMAL-SERVICE]
[2022-08-01 20:16:02.250] [nas] [debug] Sending Registration Complete
[2022-08-01 20:16:02.251] [nas] [info] Initial Registration is successful
[2022-08-01 20:16:02.251] [nas] [debug] Sending PDU Session Establishment Request
[2022-08-01 20:16:02.252] [nas] [debug] UAC access attempt is allowed for identity[0], category[MO_sig]
[2022-08-01 20:16:02.458] [nas] [debug] Configuration Update Command received
[2022-08-01 20:16:02.478] [nas] [debug] PDU Session Establishment Accept received
[2022-08-01 20:16:02.482] [nas] [info] PDU Session establishment is successful PSI[1]
[2022-08-01 20:16:02.505] [app] [info] Connection setup for PDU session[1] is successful, TUN interface[uesimtun0, 10.45.0.2] is up.
```
The Open5GS C-Plane log when executed is as follows.
```
08/01 20:16:02.218: [amf] INFO: InitialUEMessage (../src/amf/ngap-handler.c:361)
08/01 20:16:02.218: [amf] INFO: [Added] Number of gNB-UEs is now 1 (../src/amf/context.c:2087)
08/01 20:16:02.218: [amf] INFO:     RAN_UE_NGAP_ID[1] AMF_UE_NGAP_ID[1] TAC[1] CellID[0x10] (../src/amf/ngap-handler.c:497)
08/01 20:16:02.218: [amf] INFO: [suci-0-001-01-0000-0-0-0000000000] Unknown UE by SUCI (../src/amf/context.c:1409)
08/01 20:16:02.218: [amf] INFO: [Added] Number of AMF-UEs is now 1 (../src/amf/context.c:1203)
08/01 20:16:02.218: [gmm] INFO: Registration request (../src/amf/gmm-sm.c:135)
08/01 20:16:02.218: [gmm] INFO: [suci-0-001-01-0000-0-0-0000000000]    SUCI (../src/amf/gmm-handler.c:149)
08/01 20:16:02.218: [app] WARNING: Try to discover [AUSF] (../lib/sbi/path.c:114)
08/01 20:16:02.219: [amf] INFO: [3e685d3e-118b-41ed-91da-a58423211b8e] (NF-discover) NF registered (../src/amf/nnrf-handler.c:315)
08/01 20:16:02.219: [amf] INFO: [3e685d3e-118b-41ed-91da-a58423211b8e] (NF-discover) NF Profile updated (../src/amf/nnrf-handler.c:357)
08/01 20:16:02.220: [app] WARNING: Try to discover [UDM] (../lib/sbi/path.c:114)
08/01 20:16:02.221: [ausf] INFO: [3e6877b0-118b-41ed-ac84-61e8ae8d737a] (NF-discover) NF registered (../src/ausf/nnrf-handler.c:293)
08/01 20:16:02.222: [ausf] INFO: [3e6877b0-118b-41ed-ac84-61e8ae8d737a] (NF-discover) NF Profile updated (../src/ausf/nnrf-handler.c:335)
08/01 20:16:02.229: [app] WARNING: Try to discover [UDM] (../lib/sbi/path.c:114)
08/01 20:16:02.231: [amf] INFO: [3e6877b0-118b-41ed-ac84-61e8ae8d737a] (NF-discover) NF registered (../src/amf/nnrf-handler.c:315)
08/01 20:16:02.231: [amf] INFO: [3e6877b0-118b-41ed-ac84-61e8ae8d737a] (NF-discover) NF Profile updated (../src/amf/nnrf-handler.c:357)
08/01 20:16:02.236: [app] WARNING: Try to discover [PCF] (../lib/sbi/path.c:114)
08/01 20:16:02.237: [amf] INFO: [3e6e1594-118b-41ed-b12a-b7f7384a6776] (NF-discover) NF registered (../src/amf/nnrf-handler.c:315)
08/01 20:16:02.237: [amf] INFO: [3e6e1594-118b-41ed-b12a-b7f7384a6776] (NF-discover) NF Profile updated (../src/amf/nnrf-handler.c:357)
08/01 20:16:02.239: [app] WARNING: Try to discover [UDR] (../lib/sbi/path.c:114)
08/01 20:16:02.240: [pcf] INFO: [3e6e48c0-118b-41ed-bf3e-c34420bc5ec2] (NF-discover) NF registered (../src/pcf/nnrf-handler.c:295)
08/01 20:16:02.241: [pcf] INFO: [3e6e48c0-118b-41ed-bf3e-c34420bc5ec2] (NF-discover) NF Profile updated (../src/pcf/nnrf-handler.c:337)
08/01 20:16:02.448: [gmm] INFO: [imsi-001010000000000] Registration complete (../src/amf/gmm-sm.c:1116)
08/01 20:16:02.449: [amf] INFO: [imsi-001010000000000] Configuration update command (../src/amf/nas-path.c:430)
08/01 20:16:02.450: [gmm] INFO:     UTC [2022-08-01T11:16:02] Timezone[0]/DST[0] (../src/amf/gmm-build.c:531)
08/01 20:16:02.451: [gmm] INFO:     LOCAL [2022-08-01T20:16:02] Timezone[32400]/DST[0] (../src/amf/gmm-build.c:536)
08/01 20:16:02.452: [amf] INFO: [Added] Number of AMF-Sessions is now 1 (../src/amf/context.c:2101)
08/01 20:16:02.452: [gmm] INFO: UE SUPI[imsi-001010000000000] DNN[internet] S_NSSAI[SST:1 SD:0x1] (../src/amf/gmm-handler.c:1061)
08/01 20:16:02.454: [smf] INFO: [Added] Number of SMF-UEs is now 1 (../src/smf/context.c:892)
08/01 20:16:02.454: [smf] INFO: [Added] Number of SMF-Sessions is now 1 (../src/smf/context.c:2972)
08/01 20:16:02.455: [app] WARNING: Try to discover [UDM] (../lib/sbi/path.c:114)
08/01 20:16:02.457: [smf] INFO: [3e6877b0-118b-41ed-ac84-61e8ae8d737a] (NF-discover) NF registered (../src/smf/nnrf-handler.c:294)
08/01 20:16:02.457: [smf] INFO: [3e6877b0-118b-41ed-ac84-61e8ae8d737a] (NF-discover) NF Profile updated (../src/smf/nnrf-handler.c:336)
08/01 20:16:02.461: [app] WARNING: Try to discover [PCF] (../lib/sbi/path.c:114)
08/01 20:16:02.462: [smf] INFO: [3e6e1594-118b-41ed-b12a-b7f7384a6776] (NF-discover) NF registered (../src/smf/nnrf-handler.c:294)
08/01 20:16:02.463: [smf] INFO: [3e6e1594-118b-41ed-b12a-b7f7384a6776] (NF-discover) NF Profile updated (../src/smf/nnrf-handler.c:336)
08/01 20:16:02.464: [app] WARNING: Try to discover [BSF] (../lib/sbi/path.c:114)
08/01 20:16:02.465: [pcf] INFO: [3e67d5ee-118b-41ed-bdca-cbde4c328270] (NF-discover) NF registered (../src/pcf/nnrf-handler.c:295)
08/01 20:16:02.465: [pcf] INFO: [3e67d5ee-118b-41ed-bdca-cbde4c328270] (NF-discover) NF Profile updated (../src/pcf/nnrf-handler.c:337)
08/01 20:16:02.467: [smf] INFO: UE SUPI[imsi-001010000000000] DNN[internet] IPv4[10.45.0.2] IPv6[] (../src/smf/npcf-handler.c:497)
08/01 20:16:02.467: [gtp] INFO: gtp_connect() [192.168.0.114]:2152 (../lib/gtp/path.c:60)
08/01 20:16:02.468: [app] WARNING: Try to discover [AMF] (../lib/sbi/path.c:114)
08/01 20:16:02.469: [smf] INFO: [3e6c41ba-118b-41ed-8f0e-9910239c71cb] (NF-discover) NF registered (../src/smf/nnrf-handler.c:294)
08/01 20:16:02.469: [smf] INFO: [3e6c41ba-118b-41ed-8f0e-9910239c71cb] (NF-discover) NF Profile updated (../src/smf/nnrf-handler.c:336)
```
The Open5GS U-Plane1 log when executed is as follows.
```
08/01 20:16:02.480: [upf] INFO: [Added] Number of UPF-Sessions is now 1 (../src/upf/context.c:178)
08/01 20:16:02.480: [gtp] INFO: gtp_connect() [192.168.0.112]:2152 (../lib/gtp/path.c:60)
08/01 20:16:02.480: [upf] INFO: UE F-SEID[CP:0x1 UP:0x1] APN[internet] PDN-Type[1] IPv4[10.45.0.2] IPv6[] (../src/upf/context.c:397)
08/01 20:16:02.480: [upf] INFO: UE F-SEID[CP:0x1 UP:0x1] APN[internet] PDN-Type[1] IPv4[10.45.0.2] IPv6[] (../src/upf/context.c:397)
08/01 20:16:02.486: [gtp] INFO: gtp_connect() [192.168.0.131]:2152 (../lib/gtp/path.c:60)
```
The TUNnel interface `uesimtun0` is created as follows.
```
# ip addr show
...
7: uesimtun0: <POINTOPOINT,PROMISC,NOTRAILERS,UP,LOWER_UP> mtu 1400 qdisc fq_codel state UNKNOWN group default qlen 500
    link/none 
    inet 10.45.0.2/32 scope global uesimtun0
       valid_lft forever preferred_lft forever
    inet6 fe80::2bbb:2396:796e:28ac/64 scope link stable-privacy 
       valid_lft forever preferred_lft forever
...
```

<h4 id="ping_sd1">Ping google.com going through DN=10.45.0.0/16 on U-Plane1</h4>

Confirm by using `tcpdump` that the packet goes through `if=ogstun` on U-Plane1.
```
# ping google.com -I uesimtun0 -n
PING google.com (142.251.42.174) from 10.45.0.2 uesimtun0: 56(84) bytes of data.
64 bytes from 142.251.42.174: icmp_seq=1 ttl=61 time=22.7 ms
64 bytes from 142.251.42.174: icmp_seq=2 ttl=61 time=18.2 ms
64 bytes from 142.251.42.174: icmp_seq=3 ttl=61 time=18.1 ms
```
The `tcpdump` log on U-Plane1 is as follows.
```
20:17:43.059993 IP 10.45.0.2 > 142.251.42.174: ICMP echo request, id 2, seq 1, length 64
20:17:43.080002 IP 142.251.42.174 > 10.45.0.2: ICMP echo reply, id 2, seq 1, length 64
20:17:44.061849 IP 10.45.0.2 > 142.251.42.174: ICMP echo request, id 2, seq 2, length 64
20:17:44.077792 IP 142.251.42.174 > 10.45.0.2: ICMP echo reply, id 2, seq 2, length 64
20:17:45.063884 IP 10.45.0.2 > 142.251.42.174: ICMP echo request, id 2, seq 3, length 64
20:17:45.079653 IP 142.251.42.174 > 10.45.0.2: ICMP echo reply, id 2, seq 3, length 64
```
**Note. Make sure the packet does not go through U-Plane2. The UE connects to the DN of U-Plane1 based on SST:1 and SD:0x000001.**

<h3 id="run_sd2">Run UERANSIM (UE set to SST:1 and SD:0x000002)</h3>

Then the UE disconnects from gNodeB and connects to gNodeB using the configuration file for SST:1 and SD:0x000002.
Confirm that the packet goes through the DN of U-Plane2 based on SST:1 and SD:0x000002.

<h4 id="con_sd2">Start UE connected to U-Plane2 based on SST:1 and SD:0x000002</h4>

```
# ./nr-ue -c ../config/open5gs-ue-sd2.yaml 
UERANSIM v3.2.6
[2022-08-01 20:19:07.085] [nas] [info] UE switches to state [MM-DEREGISTERED/PLMN-SEARCH]
[2022-08-01 20:19:07.085] [rrc] [debug] New signal detected for cell[1], total [1] cells in coverage
[2022-08-01 20:19:07.086] [nas] [info] Selected plmn[001/01]
[2022-08-01 20:19:07.086] [rrc] [info] Selected cell plmn[001/01] tac[1] category[SUITABLE]
[2022-08-01 20:19:07.086] [nas] [info] UE switches to state [MM-DEREGISTERED/PS]
[2022-08-01 20:19:07.086] [nas] [info] UE switches to state [MM-DEREGISTERED/NORMAL-SERVICE]
[2022-08-01 20:19:07.087] [nas] [debug] Initial registration required due to [MM-DEREG-NORMAL-SERVICE]
[2022-08-01 20:19:07.087] [nas] [debug] UAC access attempt is allowed for identity[0], category[MO_sig]
[2022-08-01 20:19:07.087] [nas] [debug] Sending Initial Registration
[2022-08-01 20:19:07.088] [nas] [info] UE switches to state [MM-REGISTER-INITIATED]
[2022-08-01 20:19:07.088] [rrc] [debug] Sending RRC Setup Request
[2022-08-01 20:19:07.089] [rrc] [info] RRC connection established
[2022-08-01 20:19:07.089] [rrc] [info] UE switches to state [RRC-CONNECTED]
[2022-08-01 20:19:07.090] [nas] [info] UE switches to state [CM-CONNECTED]
[2022-08-01 20:19:07.099] [nas] [debug] Authentication Request received
[2022-08-01 20:19:07.103] [nas] [debug] Security Mode Command received
[2022-08-01 20:19:07.103] [nas] [debug] Selected integrity[2] ciphering[0]
[2022-08-01 20:19:07.113] [nas] [debug] Registration accept received
[2022-08-01 20:19:07.113] [nas] [info] UE switches to state [MM-REGISTERED/NORMAL-SERVICE]
[2022-08-01 20:19:07.114] [nas] [debug] Sending Registration Complete
[2022-08-01 20:19:07.114] [nas] [info] Initial Registration is successful
[2022-08-01 20:19:07.114] [nas] [debug] Sending PDU Session Establishment Request
[2022-08-01 20:19:07.114] [nas] [debug] UAC access attempt is allowed for identity[0], category[MO_sig]
[2022-08-01 20:19:07.318] [nas] [debug] Configuration Update Command received
[2022-08-01 20:19:07.338] [nas] [debug] PDU Session Establishment Accept received
[2022-08-01 20:19:07.343] [nas] [info] PDU Session establishment is successful PSI[1]
[2022-08-01 20:19:07.366] [app] [info] Connection setup for PDU session[1] is successful, TUN interface[uesimtun0, 10.46.0.2] is up.
```
The Open5GS C-Plane log when executed is as follows.
```
08/01 20:19:07.087: [amf] INFO: InitialUEMessage (../src/amf/ngap-handler.c:361)
08/01 20:19:07.088: [amf] INFO: [Added] Number of gNB-UEs is now 1 (../src/amf/context.c:2087)
08/01 20:19:07.088: [amf] INFO:     RAN_UE_NGAP_ID[1] AMF_UE_NGAP_ID[2] TAC[1] CellID[0x10] (../src/amf/ngap-handler.c:497)
08/01 20:19:07.089: [amf] INFO: [suci-0-001-01-0000-0-0-0000000000] known UE by SUCI (../src/amf/context.c:1407)
08/01 20:19:07.089: [gmm] INFO: Registration request (../src/amf/gmm-sm.c:135)
08/01 20:19:07.089: [gmm] INFO: [suci-0-001-01-0000-0-0-0000000000]    SUCI (../src/amf/gmm-handler.c:149)
08/01 20:19:07.092: [smf] INFO: Removed Session: UE IMSI:[imsi-001010000000000] DNN:[internet:1] IPv4:[10.45.0.2] IPv6:[] (../src/smf/context.c:1592)
08/01 20:19:07.092: [smf] INFO: [Removed] Number of SMF-Sessions is now 0 (../src/smf/context.c:2979)
08/01 20:19:07.093: [smf] INFO: [Removed] Number of SMF-UEs is now 0 (../src/smf/context.c:951)
08/01 20:19:07.093: [amf] INFO: [imsi-001010000000000:1] Release SM context [204] (../src/amf/amf-sm.c:466)
08/01 20:19:07.093: [amf] INFO: [Removed] Number of AMF-Sessions is now 0 (../src/amf/context.c:2108)
08/01 20:19:07.107: [pcf] WARNING: NF EndPoint updated [127.0.0.5:7777] (../src/pcf/npcf-handler.c:93)
08/01 20:19:07.312: [gmm] INFO: [imsi-001010000000000] Registration complete (../src/amf/gmm-sm.c:1116)
08/01 20:19:07.312: [amf] INFO: [imsi-001010000000000] Configuration update command (../src/amf/nas-path.c:430)
08/01 20:19:07.313: [gmm] INFO:     UTC [2022-08-01T11:19:07] Timezone[0]/DST[0] (../src/amf/gmm-build.c:531)
08/01 20:19:07.314: [gmm] INFO:     LOCAL [2022-08-01T20:19:07] Timezone[32400]/DST[0] (../src/amf/gmm-build.c:536)
08/01 20:19:07.315: [amf] INFO: [Added] Number of AMF-Sessions is now 1 (../src/amf/context.c:2101)
08/01 20:19:07.316: [gmm] INFO: UE SUPI[imsi-001010000000000] DNN[internet] S_NSSAI[SST:1 SD:0x2] (../src/amf/gmm-handler.c:1061)
08/01 20:19:07.318: [smf] INFO: [Added] Number of SMF-UEs is now 1 (../src/smf/context.c:892)
08/01 20:19:07.319: [smf] INFO: [Added] Number of SMF-Sessions is now 1 (../src/smf/context.c:2972)
08/01 20:19:07.320: [app] WARNING: Try to discover [UDM] (../lib/sbi/path.c:114)
08/01 20:19:07.321: [smf] INFO: [3e6877b0-118b-41ed-ac84-61e8ae8d737a] (NF-discover) NF registered (../src/smf/nnrf-handler.c:294)
08/01 20:19:07.322: [smf] INFO: [3e6877b0-118b-41ed-ac84-61e8ae8d737a] (NF-discover) NF Profile updated (../src/smf/nnrf-handler.c:336)
08/01 20:19:07.325: [app] WARNING: Try to discover [PCF] (../lib/sbi/path.c:114)
08/01 20:19:07.326: [smf] INFO: [3e6e1594-118b-41ed-b12a-b7f7384a6776] (NF-discover) NF registered (../src/smf/nnrf-handler.c:294)
08/01 20:19:07.327: [smf] INFO: [3e6e1594-118b-41ed-b12a-b7f7384a6776] (NF-discover) NF Profile updated (../src/smf/nnrf-handler.c:336)
08/01 20:19:07.330: [smf] INFO: UE SUPI[imsi-001010000000000] DNN[internet] IPv4[10.46.0.2] IPv6[] (../src/smf/npcf-handler.c:497)
08/01 20:19:07.331: [gtp] INFO: gtp_connect() [192.168.0.115]:2152 (../lib/gtp/path.c:60)
08/01 20:19:07.331: [app] WARNING: Try to discover [AMF] (../lib/sbi/path.c:114)
08/01 20:19:07.332: [smf] INFO: [3e6c41ba-118b-41ed-8f0e-9910239c71cb] (NF-discover) NF registered (../src/smf/nnrf-handler.c:294)
08/01 20:19:07.332: [smf] INFO: [3e6c41ba-118b-41ed-8f0e-9910239c71cb] (NF-discover) NF Profile updated (../src/smf/nnrf-handler.c:336)
```
The Open5GS U-Plane2 log when executed is as follows.
```
08/01 20:19:07.334: [upf] INFO: [Added] Number of UPF-Sessions is now 1 (../src/upf/context.c:178)
08/01 20:19:07.334: [gtp] INFO: gtp_connect() [192.168.0.113]:2152 (../lib/gtp/path.c:60)
08/01 20:19:07.335: [upf] INFO: UE F-SEID[CP:0x1 UP:0x1] APN[internet] PDN-Type[1] IPv4[10.46.0.2] IPv6[] (../src/upf/context.c:397)
08/01 20:19:07.335: [upf] INFO: UE F-SEID[CP:0x1 UP:0x1] APN[internet] PDN-Type[1] IPv4[10.46.0.2] IPv6[] (../src/upf/context.c:397)
08/01 20:19:07.340: [gtp] INFO: gtp_connect() [192.168.0.131]:2152 (../lib/gtp/path.c:60)
```
The TUNnel interface `uesimtun0` is created as follows.
```
# ip addr show
...
8: uesimtun0: <POINTOPOINT,PROMISC,NOTRAILERS,UP,LOWER_UP> mtu 1400 qdisc fq_codel state UNKNOWN group default qlen 500
    link/none 
    inet 10.46.0.2/32 scope global uesimtun0
       valid_lft forever preferred_lft forever
    inet6 fe80::a89a:b57e:3f23:a2c6/64 scope link stable-privacy 
       valid_lft forever preferred_lft forever
...
```

<h4 id="ping_sd2">Ping google.com going through DN=10.46.0.0/16 on U-Plane2</h4>

Confirm by using `tcpdump` that the packet goes through `if=ogstun` on U-Plane2.
```
# ping google.com -I uesimtun0 -n
PING google.com (142.251.42.174) from 10.46.0.2 uesimtun0: 56(84) bytes of data.
64 bytes from 142.251.42.174: icmp_seq=1 ttl=61 time=20.5 ms
64 bytes from 142.251.42.174: icmp_seq=2 ttl=61 time=18.5 ms
64 bytes from 142.251.42.174: icmp_seq=3 ttl=61 time=18.2 ms
```
The `tcpdump` log on U-Plane1 is as follows.
```
20:20:57.409542 IP 10.46.0.2 > 142.251.42.174: ICMP echo request, id 3, seq 1, length 64
20:20:57.426864 IP 142.251.42.174 > 10.46.0.2: ICMP echo reply, id 3, seq 1, length 64
20:20:58.410076 IP 10.46.0.2 > 142.251.42.174: ICMP echo request, id 3, seq 2, length 64
20:20:58.425859 IP 142.251.42.174 > 10.46.0.2: ICMP echo reply, id 3, seq 2, length 64
20:20:59.411688 IP 10.46.0.2 > 142.251.42.174: ICMP echo request, id 3, seq 3, length 64
20:20:59.427283 IP 142.251.42.174 > 10.46.0.2: ICMP echo reply, id 3, seq 3, length 64
```
**Note. Make sure the packet does not go through U-Plane1. The UE connects to the DN of U-Plane2 based on SST:1 and SD:0x000002.**

---
I was able to confirm the very simple configuration in which one UE connects to the UPF based on S-NSSAI. I would like to thank the excellent developers and all the contributors of Open5GS and UERANSIM.

<h2 id="changelog">Changelog (summary)</h2>

- [2022.08.01] Initial release.
