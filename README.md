# 🌐 MikroTik RouterOS 7 — Initial Configuration from Scratch

## 📋 Goal

Configure a MikroTik router from a clean/default state:

```text
                🌐 Internet
                    │
                    │
                 WAN port
                    │
             ┌──────▼──────┐
             │   MikroTik  │
             │   Router    │
             └──────┬──────┘
                    │
              LAN / Bridge
                    │
          ┌─────────┼─────────┐
          │         │         │
         💻        💻        📱
       Client     Client     Wi-Fi
```

The router will provide:

* 🌐 Internet access
* 🔌 LAN networking
* 📡 DHCP Server
* 🌍 DNS
* 🔄 NAT
* 🛡️ Basic firewall
* 📶 Wi-Fi, if supported by the device/package

---

# 1️⃣ Reset MikroTik to Factory Defaults

If you want a **completely clean configuration**, open:

```text
System → Reset Configuration
```

Enable:

```text
No Default Configuration: yes
```

Then click:

```text
Reset Configuration
```

⚠️ This removes the existing configuration.

After reboot, connect to the router using **WinBox → MAC Address**.

---

# 2️⃣ Identify Ethernet Ports

Before configuring the router, identify the interfaces.

Open:

```text
Interfaces
```

You may see:

```text
ether1
ether2
ether3
ether4
...
```

For this example:

```text
ether1 = WAN
ether2-ether5 = LAN
```

If your router has a different number of ports, adapt the configuration accordingly.

---

# 3️⃣ Create LAN Bridge

Open:

```text
Bridge → Bridge
```

Click:

```text
+
```

Create:

```text
Name: bridge-LAN
```

Click:

```text
Apply → OK
```

The bridge will represent the LAN network.

---

# 4️⃣ Add LAN Ports to the Bridge

Open:

```text
Bridge → Ports
```

Add:

```text
ether2 → bridge-LAN
ether3 → bridge-LAN
ether4 → bridge-LAN
ether5 → bridge-LAN
```

Do **not** add:

```text
ether1
```

because it will be used as WAN.

The topology is now:

```text
ether1
  │
  └── WAN

ether2 ─┐
ether3 ─┤
ether4 ─┼── bridge-LAN
ether5 ─┘
```

---

# 5️⃣ Configure LAN IP Address

Open:

```text
IP → Addresses
```

Click:

```text
+
```

Set:

```text
Address: 192.168.88.1/24
Interface: bridge-LAN
```

Click:

```text
Apply → OK
```

The router's LAN address is now:

```text
192.168.88.1
```

Network:

```text
192.168.88.0/24
```

---

# 6️⃣ Configure WAN DHCP Client

If your Internet provider/router provides an IP address automatically using DHCP:

Open:

```text
IP → DHCP Client
```

Click:

```text
+
```

Set:

```text
Interface: ether1
```

Enable:

```text
Use Peer DNS: yes
Add Default Route: yes
```

Click:

```text
Apply → OK
```

The status should eventually become:

```text
bound
```

The MikroTik has now received an IP address from the upstream network.

---

# 7️⃣ Configure DHCP Server

Open:

```text
IP → DHCP Server
```

Click:

```text
DHCP Setup
```

Select:

```text
bridge-LAN
```

Use:

```text
DHCP Address Space:
192.168.88.0/24
```

Gateway:

```text
192.168.88.1
```

Address Pool:

```text
192.168.88.10-192.168.88.254
```

DNS Server:

```text
192.168.88.1
```

Finish the wizard.

Clients connected to the LAN should now automatically receive an IP address.

Example:

```text
IP Address: 192.168.88.10
Subnet Mask: 255.255.255.0
Gateway: 192.168.88.1
DNS: 192.168.88.1
```

---

# 8️⃣ Configure DNS

Open:

```text
IP → DNS
```

Enable:

```text
Allow Remote Requests: yes
```

Add DNS servers:

```text
1.1.1.1
8.8.8.8
```

Click:

```text
Apply → OK
```

The MikroTik will now be able to act as a DNS resolver for LAN clients.

---

# 9️⃣ Configure NAT

NAT allows private LAN addresses to access the Internet through the WAN interface.

Open:

```text
IP → Firewall → NAT
```

Click:

```text
+
```

## General

Set:

```text
Chain: srcnat
Out. Interface: ether1
```

## Action

Open:

```text
Action
```

Select:

```text
Action: masquerade
```

Click:

```text
Apply → OK
```

The resulting rule should be:

```text
chain=srcnat
out-interface=ether1
action=masquerade
```

---

# 🔟 Configure Basic Firewall

Open:

```text
IP → Firewall → Filter Rules
```

Create the following rules.

## Rule 1 — Accept Established/Related

```text
Chain: input
Connection State:
established,related
Action:
accept
```

---

## Rule 2 — Drop Invalid

```text
Chain: input
Connection State:
invalid
Action:
drop
```

---

## Rule 3 — Allow LAN

```text
Chain: input
In. Interface: bridge-LAN
Action:
accept
```

---

## Rule 4 — Drop WAN Access

Create:

```text
Chain: input
In. Interface: ether1
Action:
drop
```

This prevents unsolicited access to the router from the WAN interface.

---

# 1️⃣1️⃣ Forwarding Firewall

Create the following rules under:

```text
IP → Firewall → Filter Rules
```

## Accept Established/Related

```text
Chain: forward
Connection State:
established,related
Action:
accept
```

## Drop Invalid

```text
Chain: forward
Connection State:
invalid
Action:
drop
```

## Allow LAN → WAN

```text
Chain: forward
In. Interface: bridge-LAN
Out. Interface: ether1
Action:
accept
```

## Drop Everything Else

```text
Chain: forward
Action:
drop
```

The traffic flow becomes:

```text
LAN
 │
 ▼
bridge-LAN
 │
 ▼
Firewall
 │
 ▼
NAT
 │
 ▼
ether1
 │
 ▼
Internet
```

---

# 1️⃣2️⃣ Set Router Identity

Open:

```text
System → Identity
```

Set a recognizable name:

```text
MikroTik-Router
```

For example:

```text
MikroTik-Lab
```

This makes the router easier to identify in WinBox.

---

# 1️⃣3️⃣ Configure Time

Open:

```text
System → Clock
```

Set the correct:

```text
Time Zone
```

For Ukraine:

```text
Europe/Kyiv
```

Correct time is important for:

* logs
* certificates
* monitoring
* troubleshooting
* scheduled tasks

---

# 1️⃣4️⃣ Test the Router

Open:

```text
Terminal
```

Test the Internet connection:

```bash
ping 1.1.1.1
```

Then:

```bash
ping 8.8.8.8
```

Then test DNS:

```bash
ping google.com
```

All three should work.

---

# 1️⃣5️⃣ Test From a Computer

Connect a computer to:

```text
ether2
```

Check its network configuration.

It should receive something similar to:

```text
IP:
192.168.88.10

Mask:
255.255.255.0

Gateway:
192.168.88.1

DNS:
192.168.88.1
```

Test the router:

```bash
ping 192.168.88.1
```

Test Internet connectivity:

```bash
ping 8.8.8.8
```

Test DNS:

```bash
ping google.com
```

---

# 1️⃣6️⃣ Final Network Configuration

After completing the setup:

```text
                    🌐 Internet
                         │
                         │
                      ether1
                         │
                ┌────────▼────────┐
                │    MikroTik     │
                │                 │
                │  WAN: ether1    │
                │                 │
                │  LAN:           │
                │  bridge-LAN     │
                │ 192.168.88.1    │
                └────────┬────────┘
                         │
              ┌──────────┼──────────┐
              │          │          │
           ether2      ether3     ether4
              │          │          │
              ▼          ▼          ▼
             💻         💻         📱
```

### Network

```text
LAN:
192.168.88.0/24
```

### Router

```text
192.168.88.1
```

### DHCP

```text
192.168.88.10-192.168.88.254
```

### WAN

```text
DHCP Client
ether1
```

### NAT

```text
srcnat
masquerade
out-interface=ether1
```

### DNS

```text
1.1.1.1
8.8.8.8
```

---

# 🧪 Troubleshooting

## ❌ WAN DHCP = `searching`

Check:

```text
IP → DHCP Client
```

Verify:

```text
Interface: ether1
```

Also check the physical cable and upstream device.

---

## ❌ Router can ping 8.8.8.8, but PC cannot

Check:

```text
IP → DHCP Server
```

Then verify that the PC received:

```text
192.168.88.x
```

Also check:

```text
Bridge → Ports
```

and make sure the PC's Ethernet port belongs to:

```text
bridge-LAN
```

---

## ❌ PC can ping 192.168.88.1 but not 8.8.8.8

Check:

```text
IP → Firewall → NAT
```

You need:

```text
srcnat
masquerade
ether1
```

Also check:

```text
IP → Routes
```

There should be a default route:

```text
0.0.0.0/0
```

---

## ❌ 8.8.8.8 works but google.com does not

Check:

```text
IP → DNS
```

Make sure:

```text
Allow Remote Requests: yes
```

and DNS servers are configured.

---

# ✅ Configuration Checklist

* [ ] Reset/clean MikroTik configuration
* [ ] Identify Ethernet ports
* [ ] Create `bridge-LAN`
* [ ] Add LAN ports to the bridge
* [ ] Configure `192.168.88.1/24`
* [ ] Configure WAN DHCP Client on `ether1`
* [ ] Configure DHCP Server
* [ ] Configure DNS
* [ ] Configure NAT masquerade
* [ ] Configure input firewall
* [ ] Configure forward firewall
* [ ] Set router identity
* [ ] Configure time zone
* [ ] Test router → Internet
* [ ] Test PC → router
* [ ] Test PC → Internet
* [ ] Test DNS

---

# 🎯 Result

A completely independent MikroTik router configuration:

```text
WAN
│
│ DHCP
▼
ether1
│
├── NAT
├── Firewall
└── Internet
│
▼
bridge-LAN
│
├── ether2
├── ether3
├── ether4
└── ether5
│
▼
192.168.88.0/24
│
└── DHCP
```

The configuration does **not depend on any previous MikroTik, LHGG, Vodafone, IP addresses, or settings**.
