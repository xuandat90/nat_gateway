# nat_gateway
cấu hình nat mạng nội bộ ra internet thông qua server gateway
dể sử dụng một máy chủ Ubuntu làm NAT Gateway, bạn cần thực hiện các bước sau:

1. Bật IP Forwarding
Ubuntu mặc định tắt IP forwarding, bạn cần bật nó để máy chủ có thể chuyển tiếp gói tin.

Chạy lệnh sau để bật IP forwarding tạm thời: 
echo 1 > /proc/sys/net/ipv4/ip_forward
Để bật vĩnh viễn,   file cấu hình:

sudo nano /etc/sysctl.conf

Tìm và sửa dòng:
net.ipv4.ip_forward=1
Sau đó áp dụng thay đổi:
sudo sysctl -p
2. Cấu hình IPTables để làm NAT
Giả sử:
eth0: Card mạng kết nối Internet (WAN)
eth1: Card mạng nội bộ (LAN)
Chạy lệnh sau để bật NAT:

sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
Chặn tất cả các kết nối không mong muốn:

sudo iptables -A FORWARD -i eth0 -o eth1 -m state --state RELATED,ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -i eth1 -o eth0 -j ACCEPT
Lưu lại iptables để tự động tải sau khi khởi động lại:

sudo apt install iptables-persistent -y
sudo netfilter-persistent save
3. Cấu hình DHCP (Tùy chọn)
Nếu muốn cấp IP tự động cho các máy trong mạng LAN, cài DHCP Server:

sudo apt install isc-dhcp-server -y
  file cấu hình:

sudo nano /etc/dhcp/dhcpd.conf
Thêm vào:

subnet 192.168.1.0 netmask 255.255.255.0 {
  range 192.168.1.100 192.168.1.200;
  option routers 192.168.1.1;
  option domain-name-servers 8.8.8.8, 8.8.4.4;
}
  file /etc/default/isc-dhcp-server:

INTERFACESv4="eth1"
Khởi động lại dịch vụ:

 
sudo systemctl restart isc-dhcp-server
4. Cấu hình Client
Trên các máy client, đặt Gateway là IP của Ubuntu NAT Gateway (ví dụ: 192.168.1.1).
Đặt DNS là 8.8.8.8 hoặc sử dụng dịch vụ DNS nội bộ.
5. Kiểm tra
Trên client, thử:

ping 8.8.8.8
curl ifconfig.me
Nếu có kết nối Internet, NAT Gateway đã hoạt động!
