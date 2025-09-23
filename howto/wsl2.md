# WSL2 Network settings

WSL2 has a [`mirrord`](https://learn.microsoft.com/en-us/windows/wsl/networking#mirrored-mode-networking) mode networking. This mode is the recommended mode.

It allows to connect to Linux services direct from windows host using the localhost address (`127.0.0.1`) e.g.: `ssh localhost` ssh to the WSL2 Linux instance.

It also allows access from the LAN pertaining correct firewall rules:

1. allow inbound tcp connections to the ssh port for the WSL2 Linux instance e.g. `22222` on the windows host
2. port-forward the external port `22222` to the localhost on port `22`

```netsh
netsh advfirewall firewall add rule name="sshd wsl2" dir=in localport=22222 protocol=TCP action=allow
netsh interface portproxy add v4tov4 listenport=22222 listenaddress=0.0.0.0 connectport=22 connectaddress=127.0.0.1
```




