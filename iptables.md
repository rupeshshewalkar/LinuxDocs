
### iptables

iptables tool is used to manage the Linux firewall rules.

**structure is: iptables -> Tables -> Chains -> Rules.**
	 


## IPTables has the following 4 built-in tables.

### 1. Filter Table

Filter is default table for iptables. So, if you don’t define you own table, you’ll be using filter table. Iptables’s filter table has the following built-in chains.

**INPUT chain** – Incoming to firewall. For packets coming to the local server.

**OUTPUT chain**– Outgoing from firewall. For packets generated locally and going out of the local server.

**FORWARD chain** – Packet for another NIC on the local server. For packets routed through the local server.

### 2. NAT table

Iptable’s NAT table has the following built-in chains.

PREROUTING chain – Alters packets before routing. i.e Packet translation happens immediately after the packet comes to the system (and before routing). This helps to translate the destination ip address of the packets to something that matches the routing on the local server. This is used for DNAT (destination NAT).
POSTROUTING chain – Alters packets after routing. i.e Packet translation happens when the packets are leaving the system. This helps to translate the source ip address of the packets to something that might match the routing on the desintation server. This is used for SNAT (source NAT).
OUTPUT chain – NAT for locally generated packets on the firewall.
3. Mangle table

Iptables’s Mangle table is for specialized packet alteration. This alters QOS bits in the TCP header. Mangle table has the following built-in chains.

PREROUTING chain
OUTPUT chain
FORWARD chain
INPUT chain
POSTROUTING chain
4. Raw table

Iptable’s Raw table is for configuration excemptions. Raw table has the following built-in chains.

PREROUTING chain
OUTPUT chain


Syntax Ipatable command

iptable -t <table_name> <action>  <packet pattern> -j <what to do>


<table_name> : filter(default), nat, mangle and raw 

<action> : 1)-A (--append) is append into one of the chains 
  
           INPUT:Incoming to firewall. For packets coming to the local server.
		   OUTPUT:Outgoing from firewall. For packets generated locally and going out of the local server.
		   FORWARD:packet for another NIC on the local server. For packets routed through the local server.
		   
		   2) -D (--delete) is delete a rule from a chain. specify the rule by the number or the packet pattern
		   
		   3) -L (--list)  lists the currently configured rules in the chains
		   
		   4) -F (--flush) Flushes all the rules in the current iptable chain 
		   
		   
		   
<Packet Pattern> : -s ip_address : All packets are checked for specific source IP address  like -s 192.168.1.23  or -s 192.168.1.0/24
                   -d ip_address : All packets are checked for specific destination IP address   like -d 192.168.1.23  or -d 192.168.1.0/24
				   -i interface  : All packets are checked for specific input interface i.e -i eth0 or -i eth1
				   -o interface  : All packets are checked for specific output interface i.e -i eth0 or -i eth1
				   -p port 		 : All packets are checked for specific port i.e tcp/udp/icmp
                   -sport number : All packets are checked for specific source port number i.e -sport 22 or -sport 21
				   -dport number : All packets are checked for specific destination port number i.e -dport 22 or -dport 21
				   
				   -m multiport --sports <port, port> :A variety of TCP/UDP source ports separated by commas. Unlike when -m isn't used, they do not have to be within a range.
				   -m multiport --dports <port, port> :A variety of TCP/UDP destination ports separated by commas. Unlike when -m isn't used, they do not have to be within a range.
				   -m multiport --ports <port, port>  :A variety of TCP/UDP ports separated by commas. Source and destination ports are assumed to be the same and they do not have to be within a range.
				   -m --state <state> : The most frequently tested states are:
										ESTABLISHED: The packet is part of a connection that has seen packets in both directions
										NEW: The packet is the start of a new connection
										RELATED: The packet is starting a new secondary connection. This is a common feature of such protocols such as an FTP data transfer, or an ICMP error.
										INVALID: The packet couldn't be identified. Could be due to insufficient system resources, or ICMP errors that don't match an existing data flow.

				   
<what to do> : -j DROP The packet is dropped. No message is sent to the requesting computer
               -j REJECT The packet is rejected & error message is sent to the requesting computer 
               -J ACCEPT The packet is allowed to proceed as specified with -A action: INPUT, OUTPUT OR FORWARD








EXAMPLES


Block IP traffic from an specific IP or Network.

Block from an IP

iptables -A INPUT -s 11.22.33.44 -j DROP
If you want to block only on an specific NIC

iptables -A INPUT -s 11.22.33.44 -i eth0 -j DROP
Or an specific port

iptables -A INPUT -s 11.22.33.44 -p tcp -dport 22 -j DROP
Using a Network and not only one IP

iptables -A INPUT -s 11.22.33.0/24 -j DROP
Block traffic from a specific MAC address

Suppose you want to bloc traffic some a MAC address instead of an IP address. This is handy if a DHCP server is changing the IP of the maching you want to protect from.

iptables -A INPUT -m mac --mac-source 00:11:2f:8f:f8:f8 -j DROP
Block a specific port

If all you want is to block a port, iptables can still do it.

And you can block incoming or outgoing traffic.

Block incoming traffic to a port

Suppose we need to block port 21 for incoming traffic:

iptables -A INPUT -p tcp --destination-port 21 -j DROP
But if you have two-NIC server, with one NIC facing the Internet and the other facing your local private Network, and you only one to block FTP access from outside world.

iptables -A INPUT -p tcp -i eth1 -p tcp --destination-port 21 -j DROP
In this case I'm assuming eth1 is the one facing the Internet.

You can also block a port from a specific IP address:

iptables -A INPUT -p tcp -s 22.33.44.55 --destination-port 21 -j DROP
Or even block access to a port from everywhere but a specific IP range.

iptables -A INPUT p tcp -s ! 22.33.44.0/24 --destination-port 21 -j DROP
Block outgoing traffic to a port

If you want to forbid outgoing traffic to port 25, this is useful, in the case you are running a Linux firewall for your office, and you want to stop virus from sending emails.

iptables -A FORWARD -p tcp --dport 25 -j DROP
I'm using FORWARD, as in this example the server is a firewall, but you can use OUTPUT too, to block also server self traffic.

Log traffic, before taking action

If you want to log the traffic before blocking it, for example, there is a rule in an office, where all employees have been said not to log into a given server, and you want to be sure everybody obeys the rule by blocking access to ssh port. But, at the same time you want to find the one who tried it.

iptables -A INPUT -p tcp --dport 22 -j LOG --log-prefix "dropped access to port 22"
iptables -A INPUT -p tcp --dport 22 -j DROP
You will be able to see which IP tried to access the server, but of course he couldn't.

Tips and Tricks
Because iptables executes the rules in order, if you want to change something you need to insert the rule in the specific position, or the desired effect is not going to be achieved.

List rules with numbers

iptables -nL --line-numbers
This is going to list all your rules with numbers preceding the rules. Determine where you want the inserted rule and write:

List specific chains

iptables -nL INPUT
Will list all INPUT rules.

iptables -nL FORWARD
Will list all OUTPUT rules

Insert rules

iptables -I INPUT 3 -s 10.0.0.0/8 -j ACCEPT
That is going to add a rule in position 3 of the "array"

Delete rules

iptables -D INPUT 3
That is going to remove the rule inserted above. You can also remove it, by matching it.

iptables -D INPUT -s 10.0.0.0/8 -j ACCEPT
Delete flush all rules and chains

This steps are very handy if you want to start with a completely empty and default tables:

iptables --flush
iptables --table nat --flush
iptables --table mangle --flush
iptables --delete-chain
iptables --table nat --delete-chain
iptables --table mangle --delete-chain			   
				   
				   
				   
Explain on ESTABLISHED,state & related 

iptables -A FORWARD -s 0/0 -i eth0 -d 192.168.1.58 -o eth1 -p TCP \
         --sport 1024:65535 -m multiport --dports 80,443 -j ACCEPT
 
iptables -A FORWARD -d 0/0 -o eth0 -s 192.168.1.58 -i eth1 -p TCP \
         -m state --state ESTABLISHED -j ACCEPT
		 
Here iptables is being configured to allow the firewall to accept TCP packets to be routed when they enter on interface eth0 from any IP address destined for IP address of 192.168.1.58 that is reachable via interface eth1.
The source port is in the range 1024 to 65535 and the destination ports are port 80 (www/http) and 443 (https). The return packets from 192.168.1.58 are allowed to be accepted too. 
Instead of stating the source and destination ports, you can simply allow packets related to established connections using the -m state and --state ESTABLISHED options.
