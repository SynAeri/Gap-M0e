## 2026-11-22
- Started project. Name and theme chosen
- planned learning

### planned functions
- collect security data: logs, open ports, services, disk health.
- keep data in SQLite databases
- Find unusual events with rules
- Show the status in a keyboard driven TUI and local web view(s)
- give failsafes for recoveries or monitoring on any potential network compromises.

### Command
'Sukima' <- meaning gap

## Planned learning

1. Testing command failures
```
❯ sudo -k
❯ sudo true
[sudo] password for <user>:
Sorry, try again.
[sudo] password for <user>:
❯ journalctl _COMM=sudo -n 5 --no-pager
Nov 22 20:38:25 nixos sudo[204265]: pam_unix(sudo:session): session closed for user root
Nov 22 22:37:14 nixos sudo[288763]: pam_unix(sudo:auth): authentication failure; logname=<user> uid=1000 euid=0 tty=/dev/pts/10 ...
Nov 22 22:37:19 nixos sudo[288763]: <user> : TTY=pts/10 ; PWD=<path> ; USER=root ; COMMAND=/run/current-system/sw/bin/true
Nov 22 22:37:19 nixos sudo[288763]: pam_unix(sudo:session): session opened for user root(uid=0) by <user>(uid=1000)
Nov 22 22:37:19 nixos sudo[288763]: pam_unix(sudo:session): session closed for user root
```

### Answers
Can see auth failure with a sudo authentication by <user> which is me :) uid 1000 euid=0 tty -< specifying the terminal executed?


2. Testing phone IP call:
```
❯ sudo iptables -L nixos-fw -n -v
[sudo] password for <user>: 
Chain nixos-fw (1 references)
 pkts bytes target     prot opt in     out     source               destination         
  518 31659 nixos-fw-accept  all  --  lo     *       0.0.0.0/0            0.0.0.0/0           
 121K  427M nixos-fw-accept  all  --  *      *       0.0.0.0/0            0.0.0.0/0            ctstate RELATED,ESTABLISHED
    0     0 nixos-fw-accept  tcp  --  *      *       0.0.0.0/0            0.0.0.0/0            tcp dpt:22
    0     0 nixos-fw-accept  tcp  --  *      *       0.0.0.0/0            0.0.0.0/0            tcp dpt:3000
    0     0 nixos-fw-accept  tcp  --  *      *       0.0.0.0/0            0.0.0.0/0            tcp dpt:8000
    0     0 nixos-fw-accept  tcp  --  *      *       0.0.0.0/0            0.0.0.0/0            tcp dpt:48312
    2   168 nixos-fw-accept  icmp --  *      *       0.0.0.0/0            0.0.0.0/0            icmptype 8
 7082 1284K nixos-fw-log-refuse  all  --  *      *       0.0.0.0/0            0.0.0.0/0           
```

```
❯ journalctl -k -g "refused connection" --no-pager --since "1 hour ago" -n 5
Nov 22 22:44:10 nixos kernel: refused connection: IN=wlp0s20f3 MAC=<mac> SRC=<phone-ip> DST=<laptop-ip> ... PROTO=TCP SPT=38124 DPT=80 ... SYN
...
Nov 22 22:43:36 nixos kernel: refused connection: IN=wlp0s20f3 MAC=<mac> SRC=<phone-ip> DST=<laptop-ip> ... PROTO=TCP SPT=38140 DPT=80 ... SYN
```

### Answers
My first command was for pings I didnt really have an operation to ping on phone so I just called

And it worked, as you can see its denied
- Refused connection = Firewall block
- IN=wlp0xxx Arrived via wifi for this call
- SRC = Who sent it ip
- DST = our laptop
- PROTO = The protocol used
- DPT = Destination port/ Service it seeked
- SYN = TCP connections start with a SYN Connection (not called that just SYN after -> SYN-Ack, ACK), look for thing.


3. Turn wifi on and off again:
```
❯ journalctl -f -u NetworkManager
Nov 22 14:22:11 nixos NetworkManager[1619]: <warn> ... (p2p-dev-wlp0s20f3): error setting IPv4 forwarding to '1': Success
Nov 22 16:55:00 nixos NetworkManager[1619]: <warn> ... (wlp0s20f3): link timed out.
Nov 22 16:55:00 nixos NetworkManager[1619]: <warn> ... (wlp0s20f3): Activation: failed for connection '<hotspot-ssid>'
...
Nov 22 22:55:54 nixos NetworkManager[1619]: <warn> ... (p2p-dev-wlp0s20f3): error setting IPv4 forwarding to '1': Resource temporarily unavailable
```

```
❯ journalctl -f -t wpa_supplicant
Nov 22 22:55:57 nixos wpa_supplicant[1831]: wlp0s20f3: Channel list changed: 6 GHz was enabled
Nov 22 22:55:57 nixos wpa_supplicant[1831]: wlp0s20f3: SME: Trying to authenticate with <ap-bssid> (SSID='<home-ssid>' freq=5180 MHz)
Nov 22 22:55:57 nixos wpa_supplicant[1831]: wlp0s20f3: Associated with <ap-bssid>
Nov 22 22:55:58 nixos wpa_supplicant[1831]: wlp0s20f3: WPA: Key negotiation completed with <ap-bssid> [PTK=CCMP GTK=CCMP]
Nov 22 22:55:58 nixos wpa_supplicant[1831]: wlp0s20f3: CTRL-EVENT-CONNECTED - Connection to <ap-bssid> completed
...
Nov 22 23:01:49 nixos wpa_supplicant[1831]: wlp0s20f3: CTRL-EVENT-DISCONNECTED bssid=<ap-bssid> reason=3 locally_generated=1
Nov 22 23:01:50 nixos wpa_supplicant[1831]: rfkill: WLAN soft blocked
...
Nov 22 23:01:55 nixos wpa_supplicant[1831]: wlp0s20f3: SME: Trying to authenticate with <ap-bssid> (SSID='<home-ssid>' freq=5180 MHz)
Nov 22 23:01:55 nixos wpa_supplicant[1831]: wlp0s20f3: CTRL-EVENT-CONNECTED - Connection to <ap-bssid> completed
```

### Answers

So calling journalctl I wanted to see the output for wifi connections if i turn off and on but didnt really yield much, I checked my /NetworkManager.conf. Low and behold, it only outputs Warnings
> /etc/NetworkManager/NetworkManager.conf
[logging]
level=WARN

This is a visibility gap as if I can only see warnings that means to my eyes nothing happened, I did check the wpa_supplicant and yielded much better results

as you can see:
- supplicant[xxx]: wifi: rfkill: WLAN soft blocked -> this is when the connection died
- xxxx6GHz was enabled -> Wi-Fi card starting again.

and the rest is quite digestible it automatically connected to the wifi and went back to the connection which is now completed
