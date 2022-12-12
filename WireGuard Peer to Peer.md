WireGuard Peer to Peer

1 - Create two instances on the google cloud platform peera & peerb use this link https://cloud.google.com/compute/docs/instances/custom-hostname-vm

2 - Click on ssh to connect to peera
# get root access
3 - sudo -i 
# install a generic kernel and remove the cloud kernel
4 - uname -r 
# you should see a name containing cloud in it
# installing another image
5 - apt install linux-image-amd64
# remove the old image 
6 - apt remove linux-*-cloud-*
# Enable IPv4 forwarding:
7 - nano /etc/sysctl.d/99-sysctl.conf
8 - Uncomment this line net.ipv4.ip_forward=1
# Savee by click control+ x and then click y then click enter
# Reboot system 
9 - systemctl reboot
# Get the Admin rights again
# install wireguard tools if not already installed on linux
10 - apt install wireguard-tools


# Do from 1 to 9 on peerb

Not the fun time 
On Peer A
# Generate private key
11 - wg genkey > private
# Show it  
12 - cat private 

On Peer B
# Generate private key
13 - wg genkey > private
# Show it  
14 - cat private 
# Drive Public key using Private key
15 - wg pubkey < private

On Peer A
# Add Wireguard Interface
16 - ip link add wg0 type wireguard
# Give it an ip address
17 - ip addr add 10.0.0.1/24 dev wg0
# Here is the private key you should use 
18 - wg set wg0 private-key ./private
# Set the interface up 
19 - ip link set wg0 up

On Peer B 
Do same steps from 16 to 19 change the ip to 10.0.0.2/24
20 - ip addr
# you should see something like this 
# 1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
#     link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
#     inet 127.0.0.1/8 scope host lo
#        valid_lft forever preferred_lft forever
#     inet6 ::1/128 scope host 
#        valid_lft forever preferred_lft forever
# 2: ens4: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1460 qdisc mq state UP group default qlen 1000
#     link/ether 42:01:0a:b6:00:05 brd ff:ff:ff:ff:ff:ff
#     altname enp0s4
#     inet 10.182.0.5/32 brd 10.182.0.5 scope global dynamic ens4
#        valid_lft 2347sec preferred_lft 2347sec
#     inet6 fe80::4001:aff:feb6:5/64 scope link 
#        valid_lft forever preferred_lft forever
# 3: wg0: <POINTOPOINT,NOARP,UP,LOWER_UP> mtu 1420 qdisc noqueue state UNKNOWN group default qlen 1000
#     link/none 
#     inet 10.0.0.2/24 scope global wg0
#        valid_lft forever preferred_lft forever

On Peer A  
21 - ip addr
# 1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
#     link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
#     inet 127.0.0.1/8 scope host lo
#        valid_lft forever preferred_lft forever
#     inet6 ::1/128 scope host 
#        valid_lft forever preferred_lft forever
# 2: ens4: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1460 qdisc mq state UP group default qlen 1000
#     link/ether 42:01:0a:b6:00:04 brd ff:ff:ff:ff:ff:ff
#     altname enp0s4
#     inet 10.182.0.4/32 brd 10.182.0.4 scope global dynamic ens4
#        valid_lft 3272sec preferred_lft 3272sec
#     inet6 fe80::4001:aff:feb6:4/64 scope link 
#        valid_lft forever preferred_lft forever
# 3: wg0: <POINTOPOINT,NOARP,UP,LOWER_UP> mtu 1420 qdisc noqueue state UNKNOWN group default qlen 1000
#     link/none 
#     inet 10.0.0.1/24 scope global wg0
#        valid_lft forever preferred_lft forever

# to see the info of wg intance on a Peer
# type the following on each peer
22 - wg 
# you should see something like this
On Peer A 
# public key: /Dp5hk6UnvNeU0r3xcG63C9X/q/CN0MZk8J1LQdkOlI=
# private key: (hidden)
# listening port: 58410
On Peer B
# public key: Ijrw1X0tLFcHXOEkzfEwdbB6T8IZycQuRoKaa0dO1kc=
# private key: (hidden)
# listening port: 40820

On Peer A 
# use the public key of PeerB
# you can use database to exchange Public keys
# use global ip and listening port of peer B for the endpoint global-ip:listening-port
23 - wg set wg0 peer Ijrw1X0tLFcHXOEkzfEwdbB6T8IZycQuRoKaa0dO1kc= allowed-ips 10.0.0.2/32 endpoint 10.182.0.5:40820

# use the public key of PeerA
# you can use database to exchange Public keys
# use global ip and listening port of peer B for the endpoint global-ip:listening-port
On Peer B 
24 - wg set wg0 peer /Dp5hk6UnvNeU0r3xcG63C9X/q/CN0MZk8J1LQdkOlI= allowed-ips 10.0.0.1/32 endpoint 10.182.0.4:58410

On Peer A 
# let's ping Peer B and hope it would work
25 - ping 10.0.0.2
# if you are a lucky man you should see something like this
# 64 bytes from 10.0.0.2: icmp_seq=2 ttl=64 time=0.462 ms
# 64 bytes from 10.0.0.2: icmp_seq=3 ttl=64 time=0.583 ms
# 64 bytes from 10.0.0.2: icmp_seq=4 ttl=64 time=0.454 ms
# 64 bytes from 10.0.0.2: icmp_seq=5 ttl=64 time=0.586 ms

# Now we know the handshake was successful and the peers are seeing each other inside wireguard tunnel with 
# these ips 10.0.0.2, 10.0.0.1

# type the following on each peer to check
26 - wg 

# you should see something like this
<!-- interface: wg0
  public key: /Dp5hk6UnvNeU0r3xcG63C9X/q/CN0MZk8J1LQdkOlI=
  private key: (hidden)
  listening port: 58410

peer: Ijrw1X0tLFcHXOEkzfEwdbB6T8IZycQuRoKaa0dO1kc=
  endpoint: 10.182.0.5:40820
  allowed ips: 10.0.0.2/32
  latest handshake: 1 minute, 15 seconds ago
  transfer: 1.59 KiB received, 1.68 KiB sent -->

Congratulations! 
you just finished your first handshake!!   
