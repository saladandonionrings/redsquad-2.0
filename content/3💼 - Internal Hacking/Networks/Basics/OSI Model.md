## What happens on the network when I plug my Ethernet wire ?

### **1. Physical Layer (Layer 1) Connection**

- **Electrical Signaling:** Once the cable is connected, the Ethernet NIC (Network Interface Card) on your device establishes an electrical connection with the switch or other network device on the other end of the cable.
- **Auto-Negotiation:** The devices exchange signals to determine:
    - The link speed (e.g., 10 Mbps, 100 Mbps, 1 Gbps).
    - The duplex mode (half-duplex or full-duplex).
- The link lights (LEDs) on the NIC and switch indicate the physical connection's status.

### **2. Data Link Layer (Layer 2) Activity**

#### **a. MAC Address Discovery**

- Your device's NIC has a unique **MAC (Media Access Control) address**. This address is essential for Layer 2 communication.
- The connected switch learns your device's MAC address by observing the source MAC in the frames sent by your device.

#### **b. ARP Requests**

- If your device needs to communicate with other devices on the same network, it sends an **ARP (Address Resolution Protocol) request** to find the MAC address associated with a specific IP address.
- ARP helps map IP addresses to MAC addresses for local network communication.

### **3. Network Layer (Layer 3) Configuration**

#### **a. DHCP Request (Dynamic IP Configuration)**

- Your device typically sends a **DHCP Discover** broadcast packet to find a **DHCP server** on the network.
- The DHCP server responds with an IP address, subnet mask, default gateway, and DNS servers.
- Your device uses these details to configure its network settings dynamically.

#### **b. Static IP (If Configured)**

- If your device is configured with a static IP address, it skips the DHCP process and uses the manually entered IP configuration.

### **4. Routing Setup**

- **Default Gateway:** The device sets up a route to send traffic to devices outside the local network via the default gateway (usually the router).
- The routing table is updated to include the local network and default gateway routes.

### **5. Higher-Layer Protocols Initialization**

#### **a. DNS Resolution**

- Your device communicates with the **DNS server** (specified during the DHCP process) to resolve domain names into IP addresses.
- For example, if you type "[www.example.com](http://www.example.com/)," your device queries the DNS server for its IP address.

#### **b. Network Services**

- Services like file sharing, remote access, or internet applications (e.g., web browsers) initialize and start communicating.

### **6. Switch Learning and Forwarding**

- The switch dynamically updates its MAC address table with the MAC address of your device and the port you are connected to.
- The switch uses this table to forward packets efficiently within the network.

### **7. Ongoing Communication**

- After configuration, your device can send and receive data packets:
    - **LAN Communication:** Data is sent directly to other devices on the local network.
    - **WAN Communication:** Data destined for external networks is forwarded to the router or default gateway.

### **Security Features (Optional)**

Depending on the network configuration:

- **Port Security:** The switch may have rules that restrict which MAC addresses are allowed on a specific port.
- **VLAN Assignment:** The switch might assign your device to a specific VLAN (Virtual Local Area Network) based on its port or MAC address.
- **Network Access Control (NAC):** The network may require authentication (e.g., 802.1X) before granting full access.