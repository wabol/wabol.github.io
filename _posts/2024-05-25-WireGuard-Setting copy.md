---
title: WireGuad Setting
date: 2024-05-25 11:31:00 -0700
categories: [Vault]
tags: 
Pin:
---

## How to access between two area device use WireGuard

#### Background Information
   
   - Two area device: A and B, the other device is A1, A2, B1, B2 
   - A and B all install Ubuntu 22.04
   - B has a public IP address and A no public IP address
   - A area IP is 192.168.3.x
   - B area IP is 192.168.1.x
I am trying to use the device which in B can through the LAN IP address to access the device in A.

#### Step 1: Install WireGuard
I am useing the `apt install wireguard` install it to the server. If you want to install it to the client, you can download the WireGuard software from the official website. https://www.wireguard.com/install/ 

I saw a YouTuber install it use 'pivpn' https://github.com/pivpn/pivpn

#### Step 2: Generate WireGuard key
After install WireGuard, you need to generate the WireGuard key. You can use the following command to generate the key:

```shell
sudo wg genkey | sudo tee /path/to/privatekey | sudo wg pubkey | sudo tee /path/to/publickey
```
The `wg genkey` command generates a private key and the `wg pubkey` command generates a public key.

#### Step 3: Configure WireGuard
The WireGuard configuration file example like below:
Server B:
sudo vim /etc/wireguard/wg0.conf
```shell
[Interface]
Address = 10.0.0.1/24
ListenPort = 51820
PrivateKey = Server_B_Private_key

[Peer]
PublicKey = Client_A_Public_key
AllowedIPs = 192.168.3.0/24, 10.0.0.0/24 # Just allow these two IP area use WireGuard VPN
PersistentKeepalive = 25
```
Client A :
sudo vim /etc/wireguard/wg0.conf
```shell
[Interface]
Address = 10.0.0.2/24
PrivateKey = Client_A_Private_key

[Peer]
PublicKey = Server_B_Public_key
Endpoint = my-wireguard.com:51820
AllowedIPs = 0.0.0.0/0
PersistentKeepalive = 25
```
Don't forgot add the port forwarding

#### Step 4: Start WireGuard
After configure WireGuard, you need to start the WireGuard service. You can use the following command to start the service:
```shell
sudo systemctl start wg-quick@wg0
# or you can use the following command to enable the service at system startup
# sudo wg-quick up wg0
```
Enable wg-quick@wg0 at system startup
```shell
sudo systemctl enable wg-quick@wg0
```
Check wg-quick@wg0 status
```shell
sudo systemctl status wg-quick@wg0
```
Check the WireGuard connection status
```shell
sudo wg show
```
Check IP route
```shell
ip route show
# If there no rule that you need, please add it. for example:
sudo ip route add 192.168.3.0/24 dev wg0 # This is optional
```
If there no rule that you need, please add it. for example:


#### Step 5: Confirm iptables rules
```shell
sudo iptables -A FORWARD -i wg0 -j ACCEPT
sudo iptables -A FORWARD -o wg0 -j ACCEPT
sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
```
Confirm IP forwarding
```shell
sudo sysctl -w net.ipv4.ip_forward=1
# or below command
sudo nano /etc/sysctl.conf # Modify "net.ipv4.ip_forward=1"
sudo sysctl -p # Apply the changes
```

⚠️ **Warning:** Please don't forgot set the static route in your router.
