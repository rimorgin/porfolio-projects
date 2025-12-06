# 🚀 Part 1 — FortiGate Base Network Configuration

This section details the initial setup of the **CoffeeLabs-HQ-NGFW** and **CoffeeLabs-Branch-NGFW** devices, establishing stable WAN connectivity and configuring the internal LAN networks with DHCP services.


## 1\. 🌐 Initial Device Setup and WAN Configuration

The objective here is to assign static public IP addresses to the WAN interfaces (`port1`) and configure default routes and DNS for Internet access.

### 1.1. Hostname Configuration

Hostnames were set to clearly distinguish the two firewalls.

| Site | New Hostname |
| :--- | :--- |
| HQ | **CoffeeLabs-HQ-NGFW** |
| Branch | **CoffeeLabs-Branch-NGFW** |

**HQ CLI:**

```cli
FGVMEVJ419R74QF4 # config system global 
FGVMEVJ419R74QF4 (global) # set hostname CoffeeLabs-HQ-NGFW
FGVMEVJ419R74QF4 (global) # end
CoffeeLabs-HQ-NGFW # 
```

### 1.2. WAN Interface (port1) Configuration

Both `port1` interfaces were set to **static mode** and configured with their respective public IP addresses, gateway, and DNS servers.

| Site | Interface | IP Address/Mask | Role | Alias | Access |
| :--- | :--- | :--- | :--- | :--- | :--- |
| HQ | `port1` | `10.15.20.120/25` | `wan` | `HQ-WAN-Interface` | `ping` |
| Branch | `port1` | `10.15.20.121/25` | `wan` | `Branch-WAN-Interface` | `ping` |

**HQ CLI (CoffeeLabs-HQ-NGFW):**

```cli
CoffeeLabs-HQ-NGFW # config system interface
CoffeeLabs-HQ-NGFW (interface) # edit port1
CoffeeLabs-HQ-NGFW (port1) # set mode static
CoffeeLabs-HQ-NGFW (port1) # set ip 10.15.20.120/25
CoffeeLabs-HQ-NGFW (port1) # set allowaccess ping
CoffeeLabs-HQ-NGFW (port1) # set role wan
CoffeeLabs-HQ-NGFW (port1) # set alias "HQ-WAN-Interface"
CoffeeLabs-HQ-NGFW (port1) # end
```

**Branch CLI (CoffeeLabs-Branch-NGFW):**

```cli
CoffeeLabs-Branch-NGFW # config system interface
CoffeeLabs-Branch-NGFW (interface) # edit port1
CoffeeLabs-Branch-NGFW (port1) # set mode static
CoffeeLabs-Branch-NGFW (port1) # set ip 10.15.20.121/25
CoffeeLabs-Branch-NGFW (port1) # set allowaccess ping 
CoffeeLabs-Branch-NGFW (port1) # set role wan
CoffeeLabs-Branch-NGFW (port1) # set alias "Branch-WAN-Interface"
CoffeeLabs-Branch-NGFW (port1) # end
```

### 1.3. Static Default Route and DNS Configuration

A default static route (`0.0.0.0/0`) was created to direct all non-local traffic toward the simulated Internet gateway (`10.15.20.126`), and public DNS servers were configured for name resolution.

**HQ CLI (CoffeeLabs-HQ-NGFW):**

```cli
CoffeeLabs-HQ-NGFW # config router static
CoffeeLabs-HQ-NGFW (static) # edit 1
CoffeeLabs-HQ-NGFW (1) # set gateway 10.15.20.126
CoffeeLabs-HQ-NGFW (1) # set device port1
CoffeeLabs-HQ-NGFW (1) # set dst 0.0.0.0/0
CoffeeLabs-HQ-NGFW (1) # set comment "Internet/Default Route"
CoffeeLabs-HQ-NGFW (1) # end

CoffeeLabs-HQ-NGFW # config system dns
CoffeeLabs-HQ-NGFW (dns) # set primary 1.1.1.1
CoffeeLabs-HQ-NGFW (dns) # set secondary 8.8.8.8
CoffeeLabs-HQ-NGFW (dns) # end
```

**Branch CLI (CoffeeLabs-Branch-NGFW):**

```cli
CoffeeLabs-Branch-NGFW # config router static
CoffeeLabs-Branch-NGFW (static) # edit 1
CoffeeLabs-Branch-NGFW (1) # set gateway 10.15.20.126
CoffeeLabs-Branch-NGFW (1) # set device port1
CoffeeLabs-Branch-NGFW (1) # set comment "Internet/Default Route"
CoffeeLabs-Branch-NGFW (1) # end

CoffeeLabs-Branch-NGFW # config system dns
CoffeeLabs-Branch-NGFW (dns) # set primary 1.1.1.1
CoffeeLabs-Branch-NGFW (dns) # set secondary 2.2.2.2
CoffeeLabs-Branch-NGFW (dns) # set secondary 8.8.8.8
CoffeeLabs-Branch-NGFW (dns) # end
```

### 1.4. WAN Connectivity Verification

Connectivity was verified by pinging public IP addresses and resolving domain names from the FortiGate CLI, followed by an end-to-end ping between the two firewalls' WAN interfaces.

**HQ Internet Connectivity Test:**

```cli
CoffeeLabs-HQ-NGFW # exec ping 1.1.1.1
... 0% packet loss ...

CoffeeLabs-HQ-NGFW # exec ping google.com
... 0% packet loss ...
```

**Inter-Firewall WAN Connectivity Test:**

```cli
CoffeeLabs-Branch-NGFW # exec ping 10.15.20.120
... 0% packet loss ...
```

***Result:** Successful ping between `10.15.20.121` and `10.15.20.120` confirmed Layer 3 reachability over the simulated WAN.*

-----

## 2\. 🏠 Local Area Network (LAN) Configuration

This phase involved configuring the internal interfaces and enabling the **DHCP service** on both FortiGate devices.

### 2.1. FortiGate LAN Interface (port2) Setup

The `port2` interfaces were configured as the **LAN gateway** for their respective networks and enabled for local management access.

| Site | Interface (port2) | Role | IP Address (Gateway) | Allowed Access | Alias |
| :--- | :--- | :--- | :--- | :--- | :--- |
| HQ | `port2` | `lan` | `192.168.1.254/24` | `http, https, ssh, ping` | `HQ-LAN-Interface` |
| Branch | `port2` | `lan` | `192.168.2.254/24` | `http, https, ssh, ping` | `Branch-LAN-Interface` |

**HQ CLI (CoffeeLabs-HQ-NGFW):**

```cli
CoffeeLabs-HQ-NGFW # config system interface
CoffeeLabs-HQ-NGFW (interface) # edit port2
CoffeeLabs-HQ-NGFW (port2) # set mode static
CoffeeLabs-HQ-NGFW (port2) # set ip 192.168.1.254/24
CoffeeLabs-HQ-NGFW (port2) # set allowaccess http https ssh ping
CoffeeLabs-HQ-NGFW (port2) # set role lan
CoffeeLabs-HQ-NGFW (port2) # set alias "HQ-LAN-Interface"
CoffeeLabs-HQ-NGFW (port2) # set status up
CoffeeLabs-HQ-NGFW (port2) # end
```

### 2.2. DHCP Server Configuration

A DHCP scope was configured on each `port2` interface to automatically provision client devices within the specified range, using the FortiGate's IP as the default gateway.

| Site | Interface | Domain | IP Range | Gateway | Netmask |
| :--- | :--- | :--- | :--- | :--- | :--- |
| HQ | `port2` | `coffeelabs.net` | `192.168.1.1` to `192.168.1.99` | `192.168.1.254` | `255.255.255.0` |
| Branch | `port2` | `coffeelabs.net` | `192.168.2.1` to `192.168.2.99` | `192.168.2.254` | `255.255.255.0` |

**HQ CLI (CoffeeLabs-HQ-NGFW):**

```cli
CoffeeLabs-HQ-NGFW # config system dhcp server
CoffeeLabs-HQ-NGFW (server) # edit 2
CoffeeLabs-HQ-NGFW (2) # set dns-service default
CoffeeLabs-HQ-NGFW (2) # set domain coffeelabs.net
CoffeeLabs-HQ-NGFW (2) # set ntp-service local
CoffeeLabs-HQ-NGFW (2) # set default-gateway 192.168.1.254
CoffeeLabs-HQ-NGFW (2) # set netmask 255.255.255.0
CoffeeLabs-HQ-NGFW (2) # set interface "port2"
CoffeeLabs-HQ-NGFW (2) # config ip-range
CoffeeLabs-HQ-NGFW (ip-range) # edit 1
CoffeeLabs-HQ-NGFW (1) # set start-ip 192.168.1.1
CoffeeLabs-HQ-NGFW (1) # set end-ip 192.168.1.99
CoffeeLabs-HQ-NGFW (1) # end
CoffeeLabs-HQ-NGFW (2) # set status enable
CoffeeLabs-HQ-NGFW (2) # end
```

### 2.3. End Host and Management Verification

 Now that everything is set up for LAN interface on the firewall as well as the dhcp service. Let's put our configuration to the test by verifying on the end hosts.


**CoffeeLabs-HQ-User1 IP Address Verification**


![hq-dhcp-client-ip-verify](./images/part%201/hq-dhcp-client-ip-verify.png)

**CoffeeLabs-Branch-User1 IP Address Verification**

![branch-dhcp-client-ip-verify](./images/part%201/branch-dhcp-client-ip-verify.png)


**CoffeeLabs HQ and Branch Web Management Access Verification**


<img src="./images/part%201/web-access-test.png" alt="web-access-test" width="760"/>


Verification confirmed that client devices successfully received DHCP configuration and that the FortiGate's web interface was accessible from the LAN.

  * **HQ Client IP Verification:** Client successfully acquired an IP in the `192.168.1.0/24` range (e.g., `192.168.1.1`).
  * **Branch Client IP Verification:** Client successfully acquired an IP in the `192.168.2.0/24` range (e.g., `192.168.2.1`).
  * **Web Management Access:** Confirmed successful web access to the FortiGate GUI via the LAN interface (`https://192.168.1.254`).

### 2.4. Web Server Configuration

You are correct that the web server configuration is missing from the previous write-up. I will integrate this step into the documentation, treating it as the final part of your local network setup phase (Part 1).

Here is the polished section:

-----

### 2.4. 🖥️ Web Server Configuration & Local Verification

To finalize the LAN infrastructure at the HQ site, the **CoffeeLabs-Web-Server** was configured with a **static IP address** and its default route was pointed to the **HQ FortiGate LAN interface** (`192.168.1.254`).

#### **CoffeeLabs-Web-Server Configuration**

The server was statically configured within the HQ LAN subnet.

| Component | Value |
| :--- | :--- |
| **IP Address** |`192.168.1.100/24` |
| **Default Gateway** | `192.168.1.254` |

**CLI Commands (Linux/Web Server):**

```cli
root@CoffeeLabs-Web-Server:~# ip addr add 192.168.1.100/24 brd 192.168.1.255 dev eth0
root@CoffeeLabs-Web-Server:~# ip route add default via 192.168.1.254 # Corrected to set default route for all traffic
root@CoffeeLabs-Web-Server:~# ip route list
0.0.0.0/0 via 192.168.1.254 dev eth0 
192.168.1.0/24 dev eth0 proto kernel scope link src 192.168.1.100 
```

**Verification of Gateway Reachability:**

The server successfully pinged its default gateway, confirming Layer 3 reachability to the HQ FortiGate.

```cli
root@CoffeeLabs-Web-Server:~# ping 192.168.1.254
PING 192.168.1.254 (192.168.1.254) 56(84) bytes of data.
64 bytes from 192.168.1.254: icmp_seq=1 ttl=255 time=2.28 ms
...
3 packets transmitted, 3 received, 0% packet loss
```

#### **Local Client Access Verification**

The final step for the local network setup was to confirm that a client on the HQ LAN (**CoffeeLabs-HQ-User1**) could successfully access the statically configured web server.

![web-server-access-test](./images/part%201/web-server-access-test.gif)

  * **Verification Confirmed:** **CoffeeLabs-HQ-User1** can successfully access the **CoffeeLabs-Web-Server** at `192.168.1.100`.

This completes all foundational networking and device configurations for **Part 1**. Both the HQ and Branch sites now have functional LANs, and both FortiGate NGFWs have established Internet (WAN) connectivity.

-----

The infrastructure is fully prepared. Would you like to proceed with [**Part 2: IPsec VPN Tunnel Configuration**](./Part%202:%20IPsec%20VPN%20Tunnel%20Configuration.md)?