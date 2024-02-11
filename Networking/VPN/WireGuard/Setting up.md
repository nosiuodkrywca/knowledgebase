1. Installing WireGuard (Debian/Ubuntu)

```
sudo apt update
sudo apt install wireguard
```

2. Creating private/public key pair

```
wg genkey | sudo tee /etc/wireguard/private.key
sudo chmod go= /etc/wireguard/private.key
```
```
sudo cat /etc/wireguard/private.key | wg pubkey | sudo tee /etc/wireguard/public.key
```

3. Create wg0.conf

```
sudo nano /etc/wireguard/wg0.conf
```
```
[Interface]
PrivateKey = <private key in base64>
Address = 10.243.0.1/24
ListenPort = 51820
#SaveConfig = true
```

4. Enable forwarding

```
sudo nano /etc/sysctl.conf
```
```
net.ipv4.ip_forward=1
```

or

```
sudo sysctl -w net.ipv4.ip_forward=1
```

5. Configure UFW

```
sudo nano /etc/wireguard/wg0.conf
```
```
PostUp = ufw route allow in on wg0 out on eth0
PostUp = iptables -t nat -I POSTROUTING -o eth0 -j MASQUERADE
PostUp = ip6tables -t nat -I POSTROUTING -o eth0 -j MASQUERADE
PreDown = ufw route delete allow in on wg0 out on eth0
PreDown = iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE
PreDown = ip6tables -t nat -D POSTROUTING -o eth0 -j MASQUERADE
```
```
sudo ufw allow 51820/udp
sudo ufw allow OpenSSH
```
