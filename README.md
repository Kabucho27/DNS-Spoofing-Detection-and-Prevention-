# DNS-Spoofing-Detection-and-Prevention-
Project 3rd Year

## Primary DNS SERVER Guide 
```

 *** How to Install and Configure DNS Server on Ubuntu 20 19 18 LTS
Domain Name : example.com
 Server DNS :
	- HostName : dns1
	- FQDN : dns1.example.com 
	- IP : 192.168.254.129
	- Network : 192.168.1.0/24

***Files to use
db.10 - cp db.0 to db.10 (Reverse file)
db.example.com - cp db.local to db.example.com (Forward file)
db.conf.local 
db.conf.options

	
***Step 1 - Install bind
	#apt update && upgrade
	#apt install -y bind9 bind9utils bind9-doc dnsutils

*** Forward Zone (db.example.com - cp db.local db.example.com)

***************************************************************************
zone "example.com" IN { // Domain name : example.com
    
      type master; // Primary DNS : master

     file "/etc/bind/db.example.com"; // Forward lookup file  

};

*** Forward lookup file (db.example.com)
;
; Bind data file for example.com
;
$TTL	604800
@	IN	SOA	example.com. 	root.example.com. (
			     3			; Serial
			604800			; Refresh
			86400			; Retry
			2419200			; Expire
			604800 )		; Negative Cache TTL
;--- Name Server Information
@	IN	NS	dns1.example.com.

;--- IP address of Name Server
dns1	IN	A	192.168.254.129

;-- ipv6
@	IN	AAAA	::1

;--- A - Record HostName To Ip Address  (authoritative name server)
dns1	IN	A	192.168.1.14

;--- CNAME record			(authoritative name server)

www	IN	CNAME 	dns1.example.com.
ftp	IN	CNAME	dns1.example.com.
****************************************************************************

****************************************************************************

*** Reverse Zone 

zone "254.168.192.in-addr.arpa" { //Reverse lookup name, should match your network in reverse order

     type master; // Primary DNS : master
     file "/etc/bind/db.10"; //Reverse lookup file cp db.127 db.10
};

*** Reverse lookup file

;
; Bind reverse data file for com ....... network
;
$TTL	604800
@	IN	SOA	dns1.example.com.	root.example.com. (
			3			; Serial
			604800			; Refresh
			86400			; Retry
			2419200 		; Expire
			604800 )		; Negative Cache TTL
;---Name Server Information
@	IN	NS	dns1.example.com.

;---Reverse lookup for Name Server
10	IN	PTR	dns1.example.com.

;---PTR Record IP address to HostName

******************************************************************************
******************************************************************************

***Forwarders in named.conf.options
{server ip address}
******************************************************************************
******************************************************************************

***Add nameserve for resolving
nameserver <ip address>
******************************************************************************
******************************************************************************
***Check for errors and restart bind 9
#named-checkconf
#named-checkzone (for checking reverse and forward)
#systemctl restart bind9

******************************************************************************
******************************************************************************
***Test
ping your server
nslookup
******************************************************************************
******************************************************************************
```
## Slave DNS SERVER Guide
```
***Setting Up the Slave DNS SERVER
* On Primary DNS Server

allow-transfer  { 192.168.254.130; }; //Allow Transfer of zone from the master server

also-notify { 192.168.254.130; }; //Notify slave for zone changes


*****Forward and Reverse
allow-update { none; }; //Since this is the primary DNS, it should be none.

     allow-transfer  { 172.16.10.15; }; //Allow Transfer of zone from the master server

     also-notify { 172.16.10.15; }; //Notify slave for zone changes



* On Slave DNS Server

	sudo apt-get install -y bind9 bind9utils bind9-doc dnsutils

//Forward Zone
zone "example.com" IN { //Domain name 

     type slave; //Secondary Slave DNS

     file "/var/cache/bind/db.example.com"; //Forward Zone Cache file

     masters { <primary server ip address>; }; //Master Server IP

};

//Reverse Zone
zone "20.10.172.in-addr.arpa" IN { //Reverse lookup name. Should match your network in reverse order

     type slave; // Secondary/Slave DNS

     file "/var/cache/bind/db.10"; //Reverse Zone Cache file

     masters { <primary ip address>; }; //Master Server IP

};

```
## DNSSEC Master Configuration
```
Step 1: Download and Install dnssec-tools package.
#apt-get install dnssec-tools

Step 2: Enable DNSSEC by adding the following configuration directives inside options{ }
"nano /etc/bind/named.conf.options"
	modify and add:
		dnssec-enable yes;
		dnssec-validation yes;
		dnssec-lookaside auto;


Step 3: Navigate to the location of your zone files.
	#cd /var/cache/bind
		Create a Zone Signing Key(ZSK) with the following command.
	#dnssec-keygen -a NSEC3RSASHA1 -b 2048 -n ZONE example.com
		Create a Key Signing Key(KSK) with the following command.
	dnssec-keygen -f KSK -a NSEC3RSASHA1 -b 4096 -n ZONE example.com

The directory will now have 4 keys - private/public pairs of ZSK and KSK. We have to add the public keys which contain the DNSKEY record to the zone file.


Step 4: add the public keys in (db.example.com)

$INCLUDE /etc/bind/Kexample.com.+007+22425.key
$INCLUDE /etc/bind/Kexample.com.+007+61584.key

Step 5: Sign the zone with the dnssec-signzone command.
	dnssec-signzone -3 <salt> -A -N INCREMENT -o <zonename> -t <zonefilename>

A 16 character string must be entered as the “salt”. 
The following command (head -c 1000 /dev/random | sha1sum | cut -b 1-16) outputs a random string of 16 characters which will be used as the salt.

This creates a new file named example.com.zone.signed which contains RRSIG records for each DNS record. We have to tell BIND to load this “signed” zone.

Step 6: Change the pointer file to "example.com.zone.signed"
#nano /etc/bind/named.conf.local
Change the file option inside the zone { } section.
	zone "example.com" IN {
	    type master;
	    file "example.com.zone.signed";
	    allow-transfer { 2.2.2.2; };
	    allow-update { none; };
	};

Step 6: Restart bind and 

Step 7: Check for the DNSKEY record using "dig" on the same server
	#dig DNSKEY example.com. @localhost +multiline

Step 8: Check for the presence of RRSIG records.
	#dig A example.com. @localhost +noadditional +dnssec +multiline	

```
