# Pi-hole-DNS-Failover-with-Keepalived
Probably out-dated and does not apply for normal people
Please check the comit dates, package versions, and distros used
Date of first comit is 09/13/2019

This assumes you have two seperate points of failure (servers), Pi-hole is installed (or know how to easily), and already generally understand the requirements of pi-hole, keepalived, routing/firewall rules, permissions, etc.

This tuturial will use two pi-hole (ad blocking DNS) servers on two different distros. In this case:
  - MASTER = Raspberry Pi 3 Model B+ running Raspbian 9 (Stetch) Kernel v4.19.66
  - BACKUP = Raspberry Pi 4 Model B running Manjaro 19.09 (ARM) Kernel v4.19.71
  
MASTER
  - 192.168.0.101/24 (eth0) and failover VIP 192.168.0.111/24
  - Pi-hole installed via offical curl
  - Receives all queries unless down, then failover to BACKUP
  - Sole DHCP server
  
BACKUP
  - 192.168.0.105/24 (eth0) and failover VIP 192.168.0.111/24
  - Pi-hole installed via Docker (at this time, Manjaro/Arch is not opffically supported, but you may try this from AUR https://wiki.archlinux.org/index.php/Pi-hole).
  - Receives queries when MASTER is down. Listed as secondary DNS via DHCP of MASTER
  
GATEWAY:
  - 192.168.0.1
  - DHCP disabled, no DNS relay, no static routes, no forwarding
  - DNS same as Pi-hole upstream (or 0.0.0.0), unless required differently. Defining Pi-hole DNS address may result in unreturned queries
  - IPv4 only

# Pi-hole Setup:
CONFIGURING MASTER (Raspbian Official)
  - With Pi-hole installed, configure your preferred upstream DNS servers (eg 1.1.1.1, 8.8.8.8, etc). Passwords should be same between servers for general ease of use
  - Enable listen on all interfaces, permit all origins
  - Enable DHCP server with whatever range and lease time
  - Check that server is functional in standalone
  - Edit /etc/pihole/setupVars.conf so IPV4_ADDRESS=192.168.0.111 #your VIP
  - Make any other changes in setupVars.conf you might want since installation
  - Edit /etc/dnsmasq.d/02-pihole-dhcp.conf and add dhcp-option=option:dns-server,192.168.0.111,192.168.0.105 #your VIP,your BACKUP IP
  - Alternatively , you can add a new file like 99-second-dns.conf with the same line dhcp-option=option:dns-server,192.168.0.111,192.168.0.105
  - execute$ pihole restartdns
  - Make sure everything is working still
  
CONFIGURING BACKUP (Docker official)
  - If not installed, you may be able to try this: docker run -d --name yourcontainername -p 53:53/tcp -p 53:53/udp -p 80:80 -p 443:443 -e TZ="America/Chicago" -e WEBPASSWORD=yourpassword -v "/local/pihole/path/:/etc/pihole/" -v "/local/dnsmasq.d/path/:/etc/dnsmasq.d/" --dns=127.0.0.1 --dns=1.1.1.1 --cap-add NET_ADMIN --restart=unless-stopped pihole/pihole:latest
  - Replaced with your container name, password, and local volume paths. Should leave first DNS as 127.0.0.1, should not need any other ports, but may add extra flags as you wish
  - With Pi-hole installed, configure your preferred upstream DNS servers (eg 1.1.1.1, 8.8.8.8, etc). Passwords should be same between servers for general ease of use
  - Enable listen on all interfaces, permit all origins
  - Check that server is functional in standalone
  - Edit /etc/pihole/setupVars.conf so IPV4_ADDRESS=192.168.0.111 #your VIP
  - Make any other changes in setupVars.conf you might want since installation
  - execute$ pihole restartdns (or like docker exec yourcontainername pihole restartdns)
  - Make sure everything is working still
  
# Keepalived Setup:
Configuring MASTER (latest v1.3.2)

