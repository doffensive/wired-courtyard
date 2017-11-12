# Windows Client with Cygwin

Download one: https://cygwin.com/setup-x86.exe | https://cygwin.com/setup-x86_64.exe

Run it. Select ‘Install from Internet’. Choose Auto SSH, Open SSH, Open SSL, Zip, Unzip.

Allow Cygwin Setup to resolve dependencies.

Launch Cygwin[64] Terminal from Start Menu.

```
$ ssh-host-config
$ cygrunsrv –S sshd
$ cygrunsrv --query sshd
$ ssh-keygen
$ scp .ssh/id_rsa.pub share@50.116.11.212:~/addme.pub
$ ssh share@50.116.11.212    (← Note you'll have to enter the pwd)

share@jumpbox $ cat addme.pub >> ~/.ssh/authorized_keys
share@jumpbox $ exit
```

```
$ ssh share@50.116.11.212    (← Note you should NOT have to enter pwd)

share@jumpbox $ exit
```

```
$ cygrunsrv -I AutoSSH -p /path/to/autossh -a "-M 0 -o "ServerAliveInterval 30" \
    -o "ServerAliveCountMax 3” -N -n -R 62000:localhost:22 share@50.116.11.212" \
    -e AUTOSSH_NTSERVICE=yes
$ exit
```


# Debian Linux Client

SSH RSA key prep

```
$ ssh-keygen
$ ssh-copy-id share@50.116.11.212
```

OpenSSH Server reconfiguration

```
# rm –v /etc/ssh/ssh_host_*
# dpkg-reconfigure openssh-server
# systemctl restart ssh
```

```
# apt-get update
# apt-get install –y openssh-server autossh tmux vim
```

''Now, choose systemd or rc.local method.''

```
# vim /etc/systemd/system/autossh-reverse@.service
```

Add contents to unit file, with no broken lines

```
[Unit]
Description=AutoSSH service for reverse SSH tunnel to this system from jump:62000.
After=network.target

[Service]
Environment="AUTOSSH_GATETIME=0"
ExecStart=/usr/bin/autossh -M 0 -o "ServerAliveInterval 30" \
    -o "ServerAliveCountMax 3" -N -n -R 62000:localhost:22 share@50.116.11.212
User=%i

[Install]
WantedBy=multi-user.target
```

```
# systemctl daemon-reload
# systemctl start autossh-reverse@<LOCAL_NON_ROOT_USER>.service
# systemctl enable autossh-reverse@<LOCAL_NON_ROOT_USER>.service
```

### Mod: use rc.local instead of unit file

```
# vim /etc/rc.local
```

Add to rc.local as one single line

```
/usr/bin/autossh –M 0 –o “ServerAliveInterval 30” -o “ServerAliveCountMax 3” –f –N \
    –n –R 62000:localhost:22 share@50.116.11.212 &
```

```
# chmod +x /etc/rc.local
```

# From the Jump Box

Optional

```
$ ssh-copy-id <CLIENT_USER>@localhost –p 62000
```

To Execute

```
$ ssh <CLIENT_USER>@localhost –p 62000
```
