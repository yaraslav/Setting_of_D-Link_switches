Enable AAA-mode, delete default users, credentails settings `enable-mode` and set new users, credentails and method of authentication for CLI-line (console/VTY) via a TACACS+ server and via a local base (redundant mode). 

```
configure terminal
aaa new-model
no username
username AdminUser privilege 1
username OperatorUser privilege 1
username Admin privilege 15
username Admin password Admin
username AdminUser password User
username OperatorUser password Operator
enable password level 15 Enable
enable password 12 Operator
service password-encryption
aaa authentication login CONSOLE local
aaa authentication login VTY group tacacs+ local
aaa authentication enable default enable
line console
login authentication CONSOLE
exit
line telnet
login authentication VTY
exit
line ssh
login authentication VTY
end 

```
```
configure terminal 
tacacs-server host 10.90.90.51 timeout 5 key testing123
aaa group server tacacs+ TACACS
server 10.90.90.51
aaa accounting system default start-stop group tacacs+
aaa accounting exec default start-stop group tacacs+
end
```
```
crypto key generate rsa modulus 2048
```
```
configure terminal
ip ssh server
ip ssh timeout 120
ip ssh authentication-retries 2

```

```
no ip telnet server 
```

```
configure terminal
ip http authentication aaa login-authentication WEB
ip http service-port 33333


configure terminal
crypto pki trustpoint TEST

crypto pki import TEST pem tftp: //10.90.90.51/cacert both
ssl-service-policy WEB
ssl-service-policy WEB secure-trustpoint TEST
ip http secure-server ssl-service-policy WEB




configure terminal
ip access-list TELNET-ACCESS
10 permit host 10.90.90.190 host 10.90.90.180
20 permit host 172.16.0.190 host 172.16.0.180
30 deny any any
list-remark trust host for telnet
exit 

line telnet 
access-class TELNET-ACCESS



Standard IP access list TELNET-ACCESS(ID: 1999)
    10 permit host 10.90.90.200 host 10.90.90.180
    20 permit host 172.16.0.190 host 172.16.0.180
    30 deny any any
    40 permit host 10.90.90.190 host 10.90.90.180
  trusrhost for telnet

DGS-1510-28X(config-ip-acl)#no 30
DGS-1510-28X(config-ip-acl)#deny any any 
DGS-1510-28X(config-ip-acl)#do sh access-list ip                      

Standard IP access list TELNET-ACCESS(ID: 1999)
    10 permit host 10.90.90.200 host 10.90.90.180
    20 permit host 172.16.0.190 host 172.16.0.180
    30 deny any any
    40 permit host 10.90.90.190 host 10.90.90.180
  trusrhost for telnet

DGS-1510-28X(config-ip-acl)#no 30        
DGS-1510-28X(config-ip-acl)#
DGS-1510-28X(config-ip-acl)#
DGS-1510-28X(config-ip-acl)#do sh access-list ip

Standard IP access list TELNET-ACCESS(ID: 1999)
    10 permit host 10.90.90.200 host 10.90.90.180
    20 permit host 172.16.0.190 host 172.16.0.180
    40 permit host 10.90.90.190 host 10.90.90.180
  trusrhost for telnet

DGS-1510-28X(config-ip-acl)#deny                
Incomplete command
DGS-1510-28X(config-ip-acl)#deny ?
  A.B.C.D  Source address
  any      Any source host
  host     A single host address

DGS-1510-28X(config-ip-acl)#?    
  <1-65535>    Sequence Number
  deny         Specify packets to reject
  exit         Exit from IP access-list standard configuration mode
  help         Description of the interactive help system
  list-remark  Add remarks for the specified ACL
  no           Negate a command or set its defaults
  permit       Specify packets to forward

DGS-1510-28X(config-ip-acl)#50 deny any any 
DGS-1510-28X(config-ip-acl)#do sh access-list ip

Standard IP access list TELNET-ACCESS(ID: 1999)
    10 permit host 10.90.90.200 host 10.90.90.180
    20 permit host 172.16.0.190 host 172.16.0.180
    40 permit host 10.90.90.190 host 10.90.90.180
    50 deny any any
  trusrhost for telnet




Ограничиваем доступ для определенных ip-адресов
ip access-list standard SSH-ACCESS
permit host 192.168.2.2
permit host 192.168.2.3
exit

access-class SSH-ACCESS in

```












1) Задаем пароль на *enable* и создаем пользователя и отключаем востановление пароля
```
Router(config)#no service password-recovery
```

2) Задаем *hostname* и банер
```
Router(config)# hostname R1
R1(config)#banner exec c
Enter TEXT message. End with the character 'c'.
Test banner for NetSkills
c
R1(config)#
```

`enable-mode`

4) Отключаем *HTTP* и *HTTPS*
```
Router(config)#no ip http secure-server
Router(config)#no ip http server
```

5) Ограничиваем доступ для определенных ip-адресов
```
Router(config)#ip access-list standard SSH-ACCESS
Router(config-std-nacl)#permit host 192.168.2.2
Router(config-std-nacl)#permit host 192.168.2.3
Router(config-std-nacl)#exit
Router(config)#line vty 0 4
Router(config-line)#access-class SSH-ACCESS in
```

6) Настраиваем защиту от *brute force* - максимальное количество допустимых попыток доступа и таймаут блокировки.
```
Router(config)#login delay 5
Router(config)#login block-for 60 attempts 3 within 30
```

7) Настраиваем *ААА*
```
R1(config)#aaa new-model
R1(config)#tacacs-server host 192.168.56.101
R1(config)#tacacs-server key cisco123
R1(config)#aaa authentication login default group tacacs+ local
R1(config)#aaa authorization exec default group tacacs+ local
R1(config)#aaa authorization config-command
R1(config)#aaa authorization commands 1 default group tacacs+ local
R1(config)#aaa authorization commands 15 default group tacacs+ local
R1(config)#aaa authorization console
R1(config)#aaa accounting exec default start-stop group tacacs+
R1(config)#aaa accounting commands 1 default start-stop group tacacs+
R1(config)#aaa accounting commands 15 default start-stop group tacacs+
```

Настройка *Buffered Logging* и уровня логирования
```
Router(config)# logging on
Router(config)# logging buffered 32768
Router(config)# logging buffered informational
```

2) Настройка *Syslog-сервера* и уровня логирования
```
Router(config)#logging host 192.168.1.100
Router(config)#logging trap informational
```
3) Ограничение количества логов с одного устройства
```
Router(config)#logging rate-limit all 50
```

4) Настройка времени
```
Router(config)#ntp server 192.168.1.100
Router(config)#clock timezone MSK 4
Router(config)#service timestamps log datetime msec localtime show-timezone
```
5) Резервирование оперативной памяти для консольного подключения
```
Router(config)#memory reserve console 4096
```
6) Настройка резервного копирования конфигурации на SCP-сервер
```
Router(config)#archive
Router(config-archive)#path scp://user:password@192.168.1.100/Cisco-Conf/$h-$t
Router(config-archive)#maximum 14
Router(config-archive)#time-period 1440
Router(config-archive)#write-memory
```
7) Настройка логирования команд (в случае отсутствия ААА-сервера)
```
Router(config)#archive
Router(config-archive)#log config
Router(config-archive-log-cfg)#logging enable
Router(config-archive-log-cfg)#logging size 200
Router(config-archive-log-cfg)#hidekeys
Router(config-archive-log-cfg)#notify syslog
```

8) Отключение "лишних" сервисов
```
no service tcp-small-servers echo
no service tcp-small-servers discard
no service tcp-small-servers daytime
no service tcp-small-servers chargen
no service udp-small-servers echo
no service udp-small-servers discard
no service udp-small-servers daytime
no service udp-small-servers chargen
no ip finger
no ip bootp server
no ip dhcp boot ignore
no service dhcp
no mop enabled
no ip domain-lookup
68no service pad
no ip http server
no ip http secure-server
no service config
no cdp enable
no cdp run
no lldp transmit
no lldp receive
no lldp run global
```

