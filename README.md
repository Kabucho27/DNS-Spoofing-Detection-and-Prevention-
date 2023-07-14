# DNS-Spoofing-Detection-and-Prevention-
Project 3rd Year

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
