# Proxmox VE Tips & tricks

## Redirect panel to port 443 (SSL on)
```
/sbin/iptables -F
/sbin/iptables -t nat -F
/sbin/iptables -t nat -A PREROUTING -p tcp --dport 443 -j REDIRECT --to-ports 8006
```