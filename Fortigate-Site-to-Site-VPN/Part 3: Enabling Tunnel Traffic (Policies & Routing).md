# 🚦 Part 3 — Enabling Tunnel Traffic (Policies, Address Objects & Routing)

With both sides of the Site-to-Site IPsec VPN configured, the tunnel will remain **inactive** until proper **address objects**, **firewall policies**, and **routing** are configured. This part focuses on enabling traffic to pass through the tunnel and verifying the tunnel status.

---

## 1. 🗂️ Creating Address Objects (HQ & Branch)

Before creating firewall policies, it is recommended to define **Address Objects** for the HQ and Branch LAN networks.
These objects are then referenced inside the firewall rules for cleaner, more maintainable configurations.

### **HQ NGFW — Address Objects**

| Name             | Subnet           | Purpose                                        |
| ---------------- | ---------------- | ---------------------------------------------- |
| `HQ-LAN-Subnet`     | `192.168.1.0/24` | Local LAN network used as source in policies   |
| `BRANCH-LAN-Subnet` | `192.168.2.0/24` | Remote network used as destination in policies |

### **Branch NGFW — Address Objects**

| Name             | Subnet           | Purpose                                        |
| ---------------- | ---------------- | ---------------------------------------------- |
| `BRANCH-LAN-Subnet` | `192.168.2.0/24` | Local LAN network used as source in policies   |
| `HQ-LAN-Subnet`     | `192.168.1.0/24` | Remote network used as destination in policies |

**Steps:**

1. Go to **Policy & Objects > Addresses**.
2. Click **Create New > Address**.
3. Set the name (e.g., `X-LAN-Subnet`).
4. Set **Type:** *Subnet*.
5. Enter the subnet (e.g., `192.168.X.0/24`).
6. Choose the correct **Interface** *ANY*.
7. Click **OK**.

#### **HQ-NGFW Address Objects Configuration**
<img src="./images/part%203/address-object-hq-1.png"  width="600"/>
<img src="./images/part%203/address-object-hq-2.png"  width="600"/>

#### **Branch-NGFW Address Objects Configuration**
<img src="./images/part%203/address-object-branch-1.png"  width="600"/>
<img src="./images/part%203/address-object-branch-2.png"  width="600"/>

---

## 2. 🧭 Static Route Configuration (HQ & Branch)

Static routes ensure that traffic for the remote network is forwarded into the IPsec tunnel.

### **HQ NGFW — Static Route**

| Destination      | Interface                            | Purpose                                  |
| ---------------- | --------------------------------- | ---------------------------------------- |
| `192.168.2.0/24` |   `HQ-to-Branch (tunnel interface)` | Routes Branch-bound traffic into the VPN |

### **Branch NGFW — Static Route**

| Destination      | Interface                            | Purpose                              |
| ---------------- | --------------------------------- | ------------------------------------ |
| `192.168.1.0/24` | `Branch-to-HQ (tunnel interface)` | Routes HQ-bound traffic into the VPN |

**Steps:**

1. Go to **Network > Static Routes**.
2. Click **Create New**.
3. Set **Destination** to the remote LAN.
4. Set **Interface** to the IPsec tunnel interface.
5. Click **OK**.

#### **HQ-NGFW Static Route Configuration**
<img src="./images/part%203/static-route-HQ.png"  width="600"/>

#### **Branch-NGFW Static Route Configuration**
<img src="./images/part%203/static-route-branch.png"  width="600"/>
---

## 3. 🧱 Firewall Policy Configuration (HQ & Branch)

After the address objects and routes are created, firewall policies determine which traffic is allowed through the tunnel.

### **HQ NGFW — LAN → Branch**

| Field                  | Value                   |
| ---------------------- | ----------------------- |
| **Incoming Interface** | `HQ-LAN`                |
| **Outgoing Interface** | `HQ-to-Branch (tunnel)` |
| **Source**             | `HQ-LAN-Subnet`            |
| **Destination**        | `BRANCH-LAN-Subnet`        |
| **Service**            | `ALL`                   |
| **Action**             | `ACCEPT`                |
| **NAT**                | Off                     |

### **Branch NGFW — LAN → HQ**

| Field                  | Value                   |
| ---------------------- | ----------------------- |
| **Incoming Interface** | `Branch-LAN`            |
| **Outgoing Interface** | `Branch-to-HQ (tunnel)` |
| **Source**             | `BRANCH-LAN-Subnet`        |
| **Destination**        | `HQ-LAN-Subnet`            |
| **Service**            | `ALL`                   |
| **Action**             | `ACCEPT`                |
| **NAT**                | Off                     |

**Steps:**

1. Go to **Policy & Objects > Firewall Policy**.
2. Click **Create New**.
3. Set incoming and outgoing interfaces.
4. Select the appropriate **address objects**.
5. Select **Service** *All*
6. Disable NAT.
7. Leave logging enabled.
8. Click **OK**.

#### **HQ-NGFW Firewall Policy Configuration**
<img src="./images/part%203/policy-hq-1.png"  width="600"/>
<img src="./images/part%203/policy-hq-2.png"  width="600"/>

<h4 name="issue-firewall-policy" id="issue-firewall-policy" ><b>Branch-NGFW Firewall Policy Configuration</b></h4>
<img src="./images/part%203/policy-branch-1.png"  width="600"/>
<img src="./images/part%203/policy-branch-2.png"  width="600"/>

---

## 4. 🔄 Tunnel Bring-Up Verification

Once static routes and firewall policies are in place, the tunnel should automatically initiate. 

1. Go to **Policy & Objects > Firewall Policy**.
2. Click the tunnel.
   > <img src="./images/part%203/tunnel-interface.png"  width="600"/>
3. On the menu below the tunnel, click **Show matching log**.

Now we have this logs about tunnel negotiation and association. 

<img src="./images/part%203/tunnel-log.png"  width="600"/>

There are failures in the negotiation process This is expected because the corresponding configuration on the Branch-NGFW contains one exact mismatch in the configuration, the **Pre-shared-key** which is ***CoffeeLabsNotSuperSecret*** on Branch-NGFW while ***CoffeeLabsMegaUltimateSecret*** on HQ-NGFW. When there is only one mismatch, association of tunnel will always fail. That's why it's always be keen on configuring tunnels and make sure to have the exact configuration and mirror on both sides when configuring Site-to-Site VPN. 

<img src="./images/part%203/tunnel-log-error.png"  width="600"/>

If the PSK on HQ and Branch does not match, the IPsec tunnel will fail during **Phase 1 negotiation**.
In the FortiGate logs, this typically appears as:

* `negotiate_error`
* `IPsec phase 1 error`
* `progress IPsec phase 1`
* `delete IPsec phase 1 SA`
  
---

## 5. Troubleshooting VPN tunnel

Now let's remediate this issue by going back again on Branch IPsec tunnel configuration.

1.  Head over to Branch-NFGW.
2.  Go to **VPN > IPsec Tunnels**.
3.  Double Click the row  with a Tunnel named **Branch-to-HQ**.
4.  In the **Authentication** section:
    * In the Pre-shared key Click **Change**.
    * Set **Pre-shared key** to: `CoffeeLabsMegaUltimateSecret`.
    * Click **OK**

<img src="./images/part 3/VPN-tunnel-troubleshoot.png"  width="600"/>


---

## 5. Verifying the results of VPN tunnel

Now in figure below, we can see tunnels formed both in green state and active.

<img src="./images/part 3/VPN-tunnel-success.png"  width="600"/>

Let's verify on our CoffeeLabs-Branch-User1 if it can access the CoffeeLabs-Web-Server

<img src="./images/part 3/firewall-policy-troubleshoot.png"  width="600"/>

Why can't we access it when it supposed to be accessed as intended? Tunnel is up in both ends of firewall, there's absolutely a mistake in the configuration somewhere in the adderss object or static routing or firewall policy. Typically, troubleshooting this verifying the configuration once again. from address object down to firewall policy on both firewalls! In this case, there is a misconfigured incoming interface in the firewall policy on the Branch-NGFW. See the [issue](#issue-firewall-policy) in figure. 

The incoming interface should be like this: 

<img src="./images/part 3/firewall-policy-troubleshoot-resolve.png"  width="600"/>

Because the incoming interface should be in the LAN interface not on the WAN interface. Now let's verify again on our CoffeeLabs-Branch-User1 if it can access the CoffeeLabs-Web-Server now.

<img src="./images/part 3/access-complete.gif" width="600"/>

Here is another verification that our IPsec is working properly, securely carrying traffic over the internet (simulated environment).

<img src="./images/part 3/wireshark-esp-payload.png" width="600"/>

This completes the successful deployment of the Site-to-Site IPsec VPN between the CoffeeLabs HQ and Branch networks. Thank you!
