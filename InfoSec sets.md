Enable AAA-mode, delete default users and `enable-mode` credentails setting, set new credentails and method of authentication for CLI-line. 

```
Switch#configure terminal
Switch(config)#aaa new-model
Switch(config)#no username
Switch(config)#username AdminUser privilege 1
Switch(config)#username OperatorUser privilege 1
Switch(config)#username AdminUser password Admin
Switch(config)#username OperatorUser password Operator
Switch(config)#enable password Enable
Switch(config)#service password-encryption 15
Switch(config)#aaa authentication login default local
Switch(config)#aaa authentication enable default enable
Switch(config)#line console
Switch(config-line)#login authentication default
Switch(config-line)#exit
Switch(config)#line telnet
Switch(config-line)#login authentication default
Switch(config-line)#exit
Switch(config)#line ssh
Switch(config-line)#login authentication default
Switch(config-line)#end 
Switch#
```

