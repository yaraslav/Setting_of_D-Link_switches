Settings server

Settings switches


Installation of tacacs+:
The installation is pretty much straight forward, by simply using the “apt” to retrieve and install the package from the repositories.
```
$ sudo apt-get install tacacs+
```
The above command will install and start the server service on port 49. This can be confirmed with several utilities, like lsof.
```
$ sudo lsof -i :49
COMMAND  PID USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
tac_plus 676 root    0u  IPv4 599157      0t0  TCP *:tacacs (LISTEN)
```
At this point, the tacacs is listening for connections on this machine, so we need to add some configuration on it.


**Configuring tacacs+**

If it happens that your server is running on multiple IP addresses, you should bind the tacacs+ service to a specific IP addresses. For that, the default daemon options can be modified to specify an IP address by modifying the  /etc/default/tacacs+ file. Open the file with your favorite text editor:

`$ sudo nano /etc/default/tacacs+`
This file specifies the daemon settings for the tacacs+ system. The default file will only specify the configuration file used by tacacs+.

By adding a ‘-B’ argument to this file, an IP address can be used for tacacs+ to listen.
```
#default configuration 
DAEMON_OPTS="-C /etc/tacacs+/tac_plus.conf " 
#new line binding an IP (where X.X.X.X is the IP address to listen on)
DAEMON_OPTS="-C /etc/tacacs+/tac_plus.conf -B X.X.X.X "
```
Please note that in the above we have another file ( /etc/tacacs+/tac_plus.conf  ) and this may lead to confusions.

Let’s clarify it:

`/etc/default/tacacs+`   is the file containing the configuration for the tacacs+ daemon ( the service), telling the daemon which files to load, what IP’s to listen on
`/etc/tacacs+/tac_plus.conf` is the file containing user permissions, access control lists, host keys, etc .

So, now that we know which file is for what, let’s go to the next step.

Configure the authentication options in `tac_plus.conf`
Now we will proceed to edit the tac_plus.conf file but before that we must backup to original file to refer to in case anything breaks.
```
sudo cp /etc/tacacs+/tac_plus.conf /etc/tacacs+/tac_plus.conf.old
```
Adding a key:
The first thing that needs to be created is a key for the network devices. A single key can be configured for all network devices or multiple keys can be configured per device. The option is up to the user but this guide will use a single key for simplicity’s sake.

NOTE: The same key must be configured on the Cisco Network Switch, so make sure you communicate with your network team while setting the key.
```
$ sudo nano /etc/tacacs+/tac_plus.conf
```
After adding the key, your file should look like:
```
key = "secret_TACACS+_key_here"
```
Once a key is set, we can add groups

Adding a group:
Adding groups and then assigning users to groups makes the delegation of user rights easier to handle. Below is an example of configuring the admins group with the highest privileges.
```
group = admins {
        default service = permit
        service = exec {
           priv-lvl = 15
           }
}
```
The group name is defined in the line “group = admins”,  where admins is the group name.
The `“default service = permit”` line indicates that if a command is not explicitly denied, then it is implicitly allowed.
The `“service = exec { priv-lvl = 15 }”` allows privilege level 15 in exec mode on a Cisco devices (privilege level 15 is the highest on Cisco equipment).
For a complete description of the `“priv-lvl”` attribute, please read this manual on Cisco website.

Adding an user:

Now that we have a group, let’s add an user:
```
user = firtstname.lastname {
    default service = permit
    member = admins
    login = des UD/x/quBUum56
    expires = "Sep 30 2016"
}
```

The username is defined in the line `“user = firtsname.lastname“`;
The `“default service = permit”` line indicates that if a command is not explicitly denied, then it is implicitly allowed;
The group membership is defined by the line `“member = admins”` which refer to the previous group called admins for a listing of what this user is authorized to do;
The password is defined by the line `“login = des UD/x/quBUum56“`, where:
`des` – specifies the password is encrypted ( with crypt )
`UD/x/quBUum56s` is the crypted hash of the password. To generate an encrypted password, tacacs+ has it’s own utility, called “tac_pwd“. The usage is quite simple:
```
$ sudo tac_pwd
Password to be encrypted: ASecretPassword
UD/x/quBUum56
```
The date the password ( not the account, just the password! ) will expire is set in the line < `expires = "Sep 30 2016"`>;
Note: It is best practice to use encrypted passwords in this file to add a “slight” amount of security in the event that someone should read this file. Plain text passwords are supported. If you decide to use them, however, make sure no one else has access to them. My advice is to keep up with the encrypted ones. While the encryption used is very light and could be easily cracked, it’s still better than plain text.

A good preventative measure is alsi to remove world read access on the configuration file. This can be accomplished via the following command:
```
$ sudo chmod o-r /etc/tacacs+/tac_plus.conf
$ /etc/init.d/tacacs_plus restart
```
At this point the server side is ready for connections from network devices.

```


