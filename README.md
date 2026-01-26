git config --global alias.lg "log --graph --abbrev-commit --pretty=format:'%Cred%h%Creset %C(yellow)%d%Creset %s %Cgreen(%cr)%Creset %C(bold blue)<%an>%Creset'"

sudo sysctl -w net.ipv4.ip_forward=1

sudo iptables -F FORWARD
sudo iptables -P FORWARD DROP

sudo iptables -A FORWARD -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT


sudo iptables -A FORWARD -i enp0s8 -o enp0s3 -s 192.168.56.0/24 -p tcp --dport 80  -j ACCEPT
sudo iptables -A FORWARD -i enp0s8 -o enp0s3 -s 192.168.56.0/24 -p tcp --dport 443 -j ACCEPT
sudo iptables -A FORWARD -i enp0s8 -o enp0s3 -s 192.168.56.0/24 -p udp --dport 53 -j ACCEPT
sudo iptables -A FORWARD -i enp0s8 -o enp0s3 -s 192.168.56.0/24 -p tcp --dport 53 -j ACCEPT

sudo iptables -A FORWARD -i enp0s8 -o enp0s3 -s 192.168.56.0/24 -p icmp -j ACCEPT
sudo iptables -A FORWARD -i enp0s3 -o enp0s8 -p icmp -j DROP

sudo iptables -A FORWARD -i enp0s3 -s 192.168.56.0/24 -j DROP

sudo iptables -A FORWARD -i enp0s3 -o enp0s8 -m conntrack --ctstate NEW -j DROP

sudo iptables -A FORWARD -j LOG --log-prefix "IPT FORWARD DROP: " --log-level 4
sudo iptables -A FORWARD -j DROP

# 8) NAT (inchangé)
sudo iptables -t nat -F POSTROUTING
sudo iptables -t nat -A POSTROUTING -o enp0s3 -s 192.168.56.0/24 -j MASQUERADE

#test
sudo iptables -L FORWARD -n -v --line-numbers
sudo iptables -t nat -L POSTROUTING -n -v
