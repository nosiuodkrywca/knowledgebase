# Setting up an OpenVPN connection within an unprivileged LXC container

1. After creating the container, edit the configuration file by adding a `/dev/net/tun` device:

```
nano /etc/pve/lxc/123.conf`
```

```
lxc.cgroup2.devices.allow: c 10:200 rwm
lxc.mount.entry: /dev/net dev/net none bind,create=dir
```

2. On your host machine, change the owner of your `/dev/net/tun` device to be able to access it from your unprivileged container:

```
chown 100000:100000 /dev/net/tun
```

3. Start the container, either from the GUI or by running:

```
pct start 123
```

4. Install OpenVPN:

```
sudo apt-get install openvpn
```

>For Surfshark VPN, download a list of configurations:
>```
> sudo apt-get install unzip
>cd /etc/openvpn
>sudo wget https://my.surfshark.com/vpn/api/v1/server/configurations
>sudo unzip configurations
>```
>List all available locations by using the `ls` command.

5. Save password and run OpenVPN service in the background:

```
nano pass.txt
```

```
username
password
```

```
sudo openvpn --config ~/connection.ovpn --auth-user-pass pass.txt --daemon
```

>You might need to input your credentials depending on your provider.

6. Confirm you're using a VPN connection:

```
ip a
curl ifconfig.me
```