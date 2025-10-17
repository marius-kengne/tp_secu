# 1. Activer le routage IP (indispensable)
sudo sysctl -w net.ipv4.ip_forward=1

# 2. Nettoyer les anciennes règles FORWARD
sudo iptables -F FORWARD

# 3. Définir la politique par défaut sur DROP
sudo iptables -P FORWARD DROP

sudo iptables -A FORWARD -i enp0s8 -o enp0s3 -s 192.168.56.0/24 -j ACCEPT

# HTTP / HTTPS
sudo iptables -A FORWARD -i enp0s8 -o enp0s3 -s 192.168.56.0/24 -p tcp --dport 80  -j ACCEPT
sudo iptables -A FORWARD -i enp0s8 -o enp0s3 -s 192.168.56.0/24 -p tcp --dport 443 -j ACCEPT

# DNS (UDP/TCP 53)
sudo iptables -A FORWARD -i enp0s8 -o enp0s3 -s 192.168.56.0/24 -p udp --dport 53 -j ACCEPT
sudo iptables -A FORWARD -i enp0s8 -o enp0s3 -s 192.168.56.0/24 -p tcp --dport 53 -j ACCEPT



sudo iptables -A FORWARD -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT


sudo iptables -A FORWARD -i enp0s3 -o enp0s8 -m conntrack --ctstate NEW -j DROP

sudo iptables -A FORWARD -i enp0s3 -s 192.168.56.0/24 -j DROP


sudo iptables -A FORWARD -j LOG --log-prefix "IPT FORWARD DROP: " --log-level 4
sudo iptables -A FORWARD -j DROP



# Autoriser ICMP depuis LAN → WAN (diagnostic)
sudo iptables -A FORWARD -i enp0s8 -o enp0s3 -s 192.168.56.0/24 -p icmp -j ACCEPT

# Bloquer ICMP depuis WAN → LAN (reconnaissance)
sudo iptables -A FORWARD -i enp0s3 -o enp0s8 -p icmp -j DROP



sudo iptables -t nat -A POSTROUTING -o enp0s3 -s 192.168.56.0/24 -j MASQUERADE


sudo iptables -L FORWARD -n -v


sudo iptables -t nat -L POSTROUTING -n -v


cat /proc/sys/net/ipv4/ip_forward
