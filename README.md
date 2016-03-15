# netjail

Starts an application in an new network namespace and route all its traffic through a specific OpenVPN.

Features:
	* force every kind of application into a specific connection
	* keep your network configuration as it is
	* prevent the application from connecting somewhere else
	* works without virtualization environments

"Features":
	* only works with openvpn ptp connections
	* its a Bash script and can easily be edited

Requirements:
	* Bash
	* kernel compiled with "advanced router" and "policy routing" features
	* iproute2 (http://lartc.org/howto/lartc.iproute2.html)

Usage: netjail APP

	APP is an application which should be routed only via openvpn
