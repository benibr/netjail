# netjail

Starts OpenVPN and moves the OVPN network interface into its own network namespace

## Features:
*  can force every kind of application into a specific connection
*  keep your host network configuration as it is
*  prevent the application from connecting somewhere else
*  works without virtualization environments or containers

## "Features":
*  only works with openvpn ptp connections
*  its a Bash script and can easily be edited

## Requirements:
*  Bash
*  linux kernel compiled with "advanced router" and "policy routing" features
*  iproute2 (http://lartc.org/howto/lartc.iproute2.html)

## Usecase
The main reason which lead to the development of this script was to keep its author busy while learning the capabilities of iproute2.
However, one resulting usecase is to jail a torrent application into a openvpn connection to provide anonymity.

## Usage:
```
Usage: netjail -c <openvpn.conf> [-a <application> [-u <user>]] [-n <netns name>] [-h]
	 -c file	specify the openvpn config file which should be used
	 -a application	optional: specify a application to run inside the netns
	 -u user	optional: specify a user to run the application inside the netns
	 -h		show this help

