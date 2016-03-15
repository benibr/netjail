#!/bin/bash

## functions
function show_usage {
	echo -e "Usage: app_ns.sh APP"
	echo -e "\t\tAPP is an application which should be routed only via openvpn"
}

#check priv/args
if [ "$(id -u)" != "0" ]; then
	echo "You need to be root!"
	exit
fi
if [ $# -eq 0 ]; then
	show_usage
	exit
fi

USER="user"
CONFIG="/home/$USER/openvpn/myconf.ovpn"
NS_SUB="10.123.123.0/24"
APP="$(which $1)"
NS_NAME="trnt"

CONFIGDIR=$(dirname $CONFIG )
GW_IP=${NS_SUB/0\/*/1}
NS_IP=${NS_SUB/0\/*/2}
VPN_DEV="$NS_NAME";VPN_DEV+="-vpn"
GW_DEV="$NS_NAME";GW_DEV+="-gw"
NS_DEV="$NS_NAME";NS_DEV+="-ns"
RT_TABLE="$NS_NAME";

#start VPN
openvpn --config $CONFIG --writepid /tmp/torrent-vpn.pid --dev $VPN_DEV --dev-type tun --daemon --cd "$CONFIGDIR" --route-noexec 
echo $VPN_PID
#wait for VPN_IP
while [ -z $VPN_IP ]; do
  VPN_IP=$(ip addr show dev $VPN_DEV | grep inet | awk '{print $2}' | cut -f 1 -d /)
  sleep 1
done
VPN_GW=$(ip addr show dev $VPN_DEV | grep peer | awk '{print $4}' | cut -f 1 -d /)
echo -e "\nVPN up an running with IP: $VPN_IP to endpoint: $VPN_GW\n"
## general setup
#add new routing table
echo "200 $RT_TABLE" >> /etc/iproute2/rt_tables
#add network namespace
ip netns add $NS_NAME
#add virtual interfaces
ip link add $GW_DEV type veth peer name $NS_DEV

## config namespace
#move viface to namespace
ip link set dev $NS_DEV netns $NS_NAME
#setup viface in namespace
ip netns exec $NS_NAME ip addr add dev $NS_DEV $NS_IP/24
ip netns exec $NS_NAME ip link set dev $NS_DEV up
#add routes in namespace
ip netns exec $NS_NAME ip route add $NS_SUB dev $NS_DEV
ip netns exec $NS_NAME ip route add default via $GW_IP

## config gateway
#setup gateway viface
ip addr add dev $GW_DEV $GW_IP/24
ip link set dev $GW_DEV up
#add routes to gateway viface
ip route add $NS_SUB dev $GW_DEV table $RT_TABLE
ip route add default via $VPN_GW dev $VPN_DEV table $RT_TABLE

## IP rules
#add ip rules to select routing table
ip rule add from $VPN_IP table $RT_TABLE
#ip rule add to $VPN_IP table $RT_TABLE
ip rule add from $NS_IP table $RT_TABLE
ip rule add to $NS_IP table $RT_TABLE

#enable NAT
iptables -t nat -A PREROUTING --destination $VPN_IP -j DNAT --to-destination $NS_IP
iptables -t nat -A POSTROUTING --source $NS_IP -j SNAT --to-source $VPN_IP
#start application in namespace
ip netns exec $NS_NAME su "$USER" -c "$APP"

#todo!!
kill $(cat /tmp/torrent-vpn.pid)
iptables -t nat -D PREROUTING --destination $VPN_IP -j DNAT --to-destination $NS_IP
iptables -t nat -D POSTROUTING --source $NS_IP -j SNAT --to-source $VPN_IP
#remove vifaces and namespace
ip link delete $GW_DEV
ip netns delete $NS_NAME
ised -i "/.*$RT_TABLE.*/d" /etc/iproute2/rt_tables


