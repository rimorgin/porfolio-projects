## 🔒 Part 2 — IPsec VPN Tunnel Configuration

With the base network configuration complete, this section focuses on establishing the secure **Site-to-Site IPsec VPN tunnel** between the **CoffeeLabs-HQ-NGFW** and the **CoffeeLabs-Branch-NGFW**. This part covers the full configuration of the Site-to-Site IPsec VPN on the **CoffeeLabs-HQ-NGFW** using the FortiGate web-based GUI.

---

## 1. 🔑 IPsec Phase 1 (IKE) Configuration — HQ Side

Phase 1 establishes the secure, authenticated control channel between the two FortiGate firewalls.

### **CoffeeLabs-HQ-NGFW Phase 1 Setup**

The HQ firewall acts as the initiator and points to the Branch Firewall’s WAN IP.

| Parameter                | GUI Setting                    | Rationale                                      |
| :----------------------- | :----------------------------- | :--------------------------------------------- |
| **Name**                 | `HQ-to-Branch`                 | Descriptive identifier.                        |
| **Remote Gateway**       | `10.15.20.121`                 | Branch FortiGate WAN IP.                       |
| **Interface**            | `HQ-Wan-Interface (port1)`     | Public-facing interface.                       |
| **Pre-Shared Key (PSK)** | `CoffeeLabsMegaUltimateSecret` | Strong key used for authentication.            |
| **IKE Version**          | `Version 2`                    | More advanced and secure than v1.              |
| **Proposal**             | `DES-SHA256`                   | Defines encryption and hashing.                |
| **DH Group**             | `20, 21`                       | Strong Diffie-Hellman groups for key exchange. |

---

### **Step 1: Create New Tunnel and Define Network Settings**

<img src="./images/part%202/VPN-setup-interface.png"  width="760"/>

1. Navigate to **VPN > IPsec Tunnels**.
2. Click **Create New**.
3. Set **Name** to `HQ-to-Branch`.
4. In the **Network** section:

   * Set **Remote gateway** to **Static IP Address**.
   * Set **IP Address** to `10.15.20.121`.
   * Set **Interface** to `HQ-Wan-Interface (port1)`.

<img src="./images/part%202/VPN-setup-HQ-1.png"  width="460"/>

---

### **Step 2: Define Authentication Parameters**

1. In the **Authentication** section:

   * Set **Method** to **Pre-shared Key**.
   * Enter the PSK `CoffeeLabsMegaUltimateSecret`.
   * Set **IKE Version** to **Version 2**.

<img src="./images/part%202/VPN-setup-HQ-2.png"  width="460"/>

---

### **Step 3: Configure Phase 1 Proposal**

1. Expand **Phase 1 Proposal**.
2. Set **Encryption – Authentication** to **DES–SHA256**.
3. Select **DH Groups 20 and 21**.
4. Set **Key Lifetime** to **3600 seconds**.

<img src="./images/part%202/VPN-setup-HQ-3.png"  width="460"/>
<img src="./images/part%202/VPN-setup-HQ-4.png"  width="460"/>

---

## 2. 🛡️ IPsec Phase 2 (ESP) Configuration — HQ Side

Phase 2 establishes the actual encrypted data tunnel using **ESP (Encapsulating Security Payload)**. This step defines **traffic selectors** and configures **Perfect Forward Secrecy (PFS)**.

### **CoffeeLabs-HQ-NGFW Phase 2 Setup**

The Phase 2 selector ensures that traffic from **HQ LAN (`192.168.1.0/24`)** to **Branch LAN (`192.168.2.0/24`)** passes through the encrypted tunnel.

| Parameter          | GUI Setting      | Rationale                      |
| :----------------- | :--------------- | :----------------------------- |
| **Local Address**  | `192.168.1.0/24` | HQ internal network.           |
| **Remote Address** | `192.168.2.0/24` | Branch internal network.       |
| **Encapsulation**  | `Tunnel Mode`    | Required for site-to-site VPN. |
| **PFS**            | `Enable`         | Strengthens key security.      |
| **Key Lifetime**   | `3600` seconds   | For the data channel.          |

---

### **Step 1: Create New Phase 2 Selector**

1. Expand **Phase 2 Selectors** → click **Create New**.
2. Set **Name** to `HQ-LAN-TO-BRANCH-LAN`.
3. Ensure **Encapsulation** is set to **Tunnel Mode**.

---

### **Step 2: Define Traffic Selectors**

1. **Local Address:** `192.168.1.0/24`
2. **Remote Address:** `192.168.2.0/24`

<img src="./images/part%202/VPN-setup-HQ-5.png"  width="460"/>

---

### **Step 3: Configure Phase 2 Proposal**

1. Under **Advanced**:

   * Set **Encryption – Authentication:** DES–SHA256.
   * Enable **Perfect Forward Secrecy (PFS)**.
   * Select **DH Groups 20 and 21**.
   * Set **Key Lifetime** to **3600 seconds**.

<img src="./images/part%202/VPN-setup-HQ-6.png"  width="460"/>
<img src="./images/part%202/VPN-setup-HQ-7.png"  width="460"/>

---

## 3. 🔑 IPsec Phase 1 (IKE) Configuration — Branch Side

Phase 1 settings on the Branch firewall must match HQ—**except in this example, a mismatch is intentionally left for later troubleshooting**.

### **CoffeeLabs-Branch-NGFW Phase 1 Setup**

| Parameter                | GUI Setting                                         | Rationale                |
| :----------------------- | :-------------------------------------------------- | :----------------------- |
| **Name**                 | `Branch-to-HQ`                                      | Descriptive identifier.  |
| **Remote Gateway**       | `10.15.20.120`                                      | HQ WAN IP.               |
| **Interface**            | `Branch-Wan-Interface (port1)`                      | Public-facing interface. |
| **Pre-Shared Key (PSK)** | `CoffeeLabsMegaUltimateSecret` → **Mismatch noted** | Must match HQ PSK.       |
| **IKE Version**          | `Version 2`                                         | Modern standard.         |
| **Proposal**             | `DES-SHA256`                                        | Must match HQ.           |
| **DH Group**             | `20, 21`                                            | Must match HQ.           |

---

### **Step 1: Create New Tunnel and Define Network Settings**

1. Go to **VPN > IPsec Tunnels**.
2. Click **Create New**.
3. Name it `Branch-to-HQ`.
4. Set:

   * **Remote Gateway:** `10.15.20.120`
   * **Interface:** `Branch-Wan-Interface (port1)`

<img src="./images/part%202/VPN-setup-Branch-1.png"  width="460"/>

---

### **Step 2: Define Authentication Parameters**

1. Set **Method:** Pre-shared Key.
2. Enter the PSK: `CoffeeLabsNotSuperSecret`
   ⚠️ *Notice: This does **not match** the HQ PSK — intentional for troubleshooting later.*
3. Set **IKE Version:** Version 2.

<img src="./images/part%202/VPN-setup-Branch-2.png"  width="460"/>

---

### **Step 3: Configure Phase 1 Proposal**

1. Expand **Phase 1 Proposal**:

   * **Encryption – Authentication:** DES–SHA256
   * **DH Groups:** 20 and 21
   * **Key Lifetime:** 3600 seconds

<img src="./images/part%202/VPN-setup-Branch-3.png"  width="460"/>
<img src="./images/part%202/VPN-setup-Branch-4.png"  width="460"/>

---

## 4. 🛡️ IPsec Phase 2 (ESP) Configuration — Branch Side

### **CoffeeLabs-Branch-NGFW Phase 2 Setup**

Defines encrypted traffic flow from **Branch LAN (`192.168.2.0/24`)** to **HQ LAN (`192.168.1.0/24`)**.

| Parameter         | GUI Setting      | Rationale              |
| :---------------- | :--------------- | :--------------------- |
| **Local Subnet**  | `192.168.2.0/24` | Branch LAN.            |
| **Remote Subnet** | `192.168.1.0/24` | HQ LAN.                |
| **Encapsulation** | `Tunnel Mode`    | Required.              |
| **PFS**           | `Enable`         | Uses DH groups 20, 21. |
| **Key Lifetime**  | `3600` seconds   | Data channel lifetime. |

---

### **Step 1: Create New Phase 2 Selector**

1. Expand **Phase 2 Selectors** → click **Create New**.
2. Name it `Branch-LAN-TO-HQ-LAN`.
3. Confirm **Tunnel Mode**.

---

### **Step 2: Define Traffic Selectors**

1. **Local Address:** `192.168.2.0/24`
2. **Remote Address:** `192.168.1.0/24`

<img src="./images/part%202/VPN-setup-Branch-5.png"  width="460"/>

---

### **Step 3: Configure Phase 2 Proposal**

1. Under **Advanced**:

   * Encryption–Authentication: **DES–SHA256**
   * Enable **PFS**
   * DH Groups **20, 21**
   * Key Lifetime: **3600 seconds**

<img src="./images/part%202/VPN-setup-Branch-6.png"  width="460"/>
<img src="./images/part%202/VPN-setup-Branch-7.png"  width="460"/>

Click **OK** to finalize.

---

## ⛔ Part 3 — Initial Tunnel Status (Expected: Inactive)

After completing the configuration on both firewalls, the tunnel will initially show as **Inactive**. This is normal because **no firewall policies or routes have been configured yet**, so the devices cannot begin negotiating the tunnel.

<img src="./images/part%202/VPN-tunnels-inactive.png"  width="850"/>

The next step is to configure the routing and firewall policies required for tunnel establishment and traffic flow. Would you like to proceed with [**Part 3: Enabling Tunnel Traffic (Policies & Routing)**](./Part%203:%20Enabling%20Tunnel%20Traffic%20(Policies%20&%20Routing).md)?

---

