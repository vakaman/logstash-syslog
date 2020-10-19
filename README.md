Logstash server configuration

```
# Fist we need to create a folder, to save all logs
mkdir /var/log/logstash

# Adjust permission
chmod -R 1000:0 /var/log/logstash

# Up logstash
docker-compose up -d 
```
Mikrotik configuration CGNAT log

```
# Configure remote logging server logstash
/system logging action
add bsd-syslog=yes remote=IP-LOGSTASH-SERVER remote-port=5514 src-address=CGNAT-IP syslog-facility=syslog

# Active logging script
/system logging
add action=remote prefix=CGNAT_LOG topics=info

# Firewal filter in output interface
/ip firewall filter
add action=log chain=forward comment=CGNAT_LOG connection-nat-state=srcnat log-prefix=CGNAT_LOG out-interface=OUTPUT-INTERFACE-UPLINK
```



Mikrotik configuration Prefix Delegation log

```
# Add log_ipv6 on-up
/ppp profile
add change-tcp-mss=yes dhcpv6-pd-pool=Pool-Clientes-PD insert-queue-before=first local-address=BRAS-IP name=IPV6 on-up=log_ipv6 only-one=yes remote-address=Pool-Clientes \
    remote-ipv6-prefix-pool=Pool-Cliente-Tunel use-compression=no use-encryption=no use-mpls=no use-upnp=no

# Configure remote logging server logstash
/system logging action
add bsd-syslog=yes name=remoteV6 remote=IP-LOGSTASH-SERVER remote-port=5515 src-address=x.x.x.x syslog-facility=syslog target=remote

# Active logging script
/system logging 
add action=remoteV6 topics=info,script

# Add script
/system script
add dont-require-permissions=no name=log_ipv6 owner=admin policy=\
    ftp,reboot,read,write,policy,test,password,sniff,sensitive,romon source=":global DateTime ([ / system clock get date ] . \"\
    \_\" . [ / system clock get time ]);\r\
    \n\r\
    \n:local interfaceName [/interface get \$interface name]; \r\
    \ndelay 60;\r\
    \n:local PrefixoIPv6 [/ipv6 dhcp-server binding get value-name=address [find server=\$interfaceName]];\r\
    \n:log info \"\$DateTime \$interfaceName \$PrefixoIPv6\""
```
