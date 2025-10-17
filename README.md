# 0) Routage activé
sudo sysctl -w net.ipv4.ip_forward=1

# 1) Remise à zéro FORWARD
sudo iptables -F FORWARD
sudo iptables -P FORWARD DROP

# 2) Autoriser d’abord les retours de connexions
sudo iptables -A FORWARD -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT

# 3) Autoriser SEULEMENT les services nécessaires LAN -> WAN
sudo iptables -A FORWARD -i enp0s8 -o enp0s3 -s 192.168.56.0/24 -p tcp --dport 80  -j ACCEPT
sudo iptables -A FORWARD -i enp0s8 -o enp0s3 -s 192.168.56.0/24 -p tcp --dport 443 -j ACCEPT
sudo iptables -A FORWARD -i enp0s8 -o enp0s3 -s 192.168.56.0/24 -p udp --dport 53 -j ACCEPT
sudo iptables -A FORWARD -i enp0s8 -o enp0s3 -s 192.168.56.0/24 -p tcp --dport 53 -j ACCEPT

# 4) ICMP: autoriser LAN -> WAN, bloquer WAN -> LAN
sudo iptables -A FORWARD -i enp0s8 -o enp0s3 -s 192.168.56.0/24 -p icmp -j ACCEPT
sudo iptables -A FORWARD -i enp0s3 -o enp0s8 -p icmp -j DROP

# 5) Anti-spoof: empêcher des sources LAN vues côté WAN
sudo iptables -A FORWARD -i enp0s3 -s 192.168.56.0/24 -j DROP

# 6) Bloquer les paquets NEW non sollicités venant du WAN vers le LAN
sudo iptables -A FORWARD -i enp0s3 -o enp0s8 -m conntrack --ctstate NEW -j DROP

# 7) Fin de chaîne: LOG puis DROP (catch-all)
sudo iptables -A FORWARD -j LOG --log-prefix "IPT FORWARD DROP: " --log-level 4
sudo iptables -A FORWARD -j DROP

# 8) NAT (inchangé)
sudo iptables -t nat -F POSTROUTING
sudo iptables -t nat -A POSTROUTING -o enp0s3 -s 192.168.56.0/24 -j MASQUERADE
