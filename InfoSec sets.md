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
aaa authentication login WEB group tacacs+ local
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

```
```
configure terminal
ip access-list VTY-ACCESS
10 permit host 10.90.90.190 host 10.90.90.180
20 permit host 172.16.0.190 host 172.16.0.180
30 deny any any
list-remark trust host for telnet
exit 

line telnet 
access-class VTY-ACCESS
end 

```





