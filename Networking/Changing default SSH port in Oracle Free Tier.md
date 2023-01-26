# Changing SSH port on Oracle Always Free Compute instance

1. Open the port in Oracle Virtual Cloud Network via GUI

2. Edit `/etc/ssh/sshd_config`

3. Change port to a desired one

    > Note: it is recommended to not change the default port before making sure the new config is working fine. Instead, you should add the new port, then delete the default one afterwards.

4. Run `sudo iptables -I INPUT 6 -m state --state NEW -p tcp --dport **_PORT_** -j ACCEPT`

5. Run `sudo netfilter-persistent save`

6. Run `sudo systemctl restart sshd`