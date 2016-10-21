#!/bin/bash
# Run this only on ppp up.

# IP Tables executable
IPT="/sbin/iptables"

# Allowable hosts file
HOSTS="/etc/sailstudio/sailstudioservers.conf"

# DNS Servers
NAMESERVER="$(grep 'nameserver' /etc/resolv.conf | awk '{ print $2}')"

# SailStudio Servers
SAILSTUDIOSERVERS="$(cat $HOSTS | awk '{print $1}')"

#echo $SAILSTUDIOSERVERS

echo "flush iptable rules"
$IPT -F
$IPT -X
$IPT -t nat -F
$IPT -t nat -X
$IPT -t mangle -F
$IPT -t mangle -X

echo "Set default policy to 'DROP'"
$IPT -P INPUT   DROP
$IPT -P FORWARD DROP
$IPT -P OUTPUT  DROP

# Allow DNS lookups (though in fact we have done this already, so is this necessary??
for ip in $NAMESERVER
do
	echo "Allowing DNS lookups (tcp, udp port 53) to server '$ip'"
	$IPT -A OUTPUT -p udp -d $ip --dport 53 -m state --state NEW,ESTABLISHED -j ACCEPT
	$IPT -A INPUT  -p udp -s $ip --sport 53 -m state --state ESTABLISHED     -j ACCEPT
	$IPT -A OUTPUT -p tcp -d $ip --dport 53 -m state --state NEW,ESTABLISHED -j ACCEPT
	$IPT -A INPUT  -p tcp -s $ip --sport 53 -m state --state ESTABLISHED     -j ACCEPT
done

echo "allow all and everything on localhost AND eth0 and wlan0"
$IPT -A INPUT -i lo -j ACCEPT
$IPT -A OUTPUT -o lo -j ACCEPT
$IPT -A INPUT -i eth0 -j ACCEPT
$IPT -A OUTPUT -o eth0 -j ACCEPT
$IPT -A INPUT -i wlan0 -j ACCEPT
$IPT -A OUTPUT -o wlan0 -j ACCEPT


# Allow connections to all sailstudio servers
for host in $SAILSTUDIOSERVERS
do
	echo $host
	ip="$(host -W 10 $host | awk '{ print $4}')"

	echo "Allow connection to '$ip'"
	$IPT -A OUTPUT -p tcp -d "$ip" -m state --state NEW,ESTABLISHED -j ACCEPT
	$IPT -A INPUT  -p tcp -s "$ip" -m state --state ESTABLISHED     -j ACCEPT
done
