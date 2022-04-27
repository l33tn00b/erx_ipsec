# erx_ipsec
IPSec connection ER-X to Ubuntu host

# Edgerouter
- Settings:  
![Settings EdgeRouter](https://github.com/l33tn00b/erx_ipsec/blob/main/settings_er.png)
- Add Route: Routing -> Tab: Routes -> Button: Add Static Route, Type: Gateway

# Ubuntu Host
- need to have strongswan installed
- create config file on server with public ip named `<conn name>.conf`
  ```
  conn <name>
          type=tunnel
          # do not auto-start but wait for incoming connection because
          # peer is behind nat and dyn ip
          auto=add
          # set parameters according to settings on ER
          esp=aes128-sha1-modp2048
          # need to have a <conn name>.secrets file in the
          # same directory
          authby=secret
          # because peer has dynamic ip we need
          # to allow any
          left=%any
          leftid=
          # this is deprecated but...
          leftnexthop=%defaultroute
          # network behind peer
          leftsubnet=192.168.3.0/24
          # this is our  public ip
          right=<server ip>
          # whatever net we'd like to provide
          rightsubnet=10.8.5.0/24
          # perfect forward secrecy, please
          pfs=yes        
  ```
- since this is a bogus net on our server, add an appropriate ip to the loopback interface
  `ip addr add 10.8.5.1/24 dev lo `
- add route: `ip route add 192.168.3.0/24 via 10.8.5.1`
- (maybe)we should think about using vti/gre so we don't mess up routing with clients having dialled in via openvpn): https://docs.strongswan.org/strongswan-docs/5.9/features/routeBasedVpn.html because right now, we do `-A POSTROUTING -s 10.8.0.0/8 -o eth0 -j MASQUERADE   ` in /etc/ufw/before.rules. Thus, anything going out there will get natted and will not match the defined security policy of 10.8.5.0/25 == 192.168.3.0
- trouble is, packets (e.g. ping) get routed out eth0 (with a public ip) to a private destination (coming in from tun0, do a tcpdump -i tun0 to see incoming, do a tcpdump -i eth0 to see outgoing, unencapsulated/encrypted (ping) packets)
- take care of allowing traffic from/to interfaces in ufw (/etc/ufw/before.rules): "DEFAULT_FORWARD_POLICY variable in /etc/default/ufw from a value of "DROP" to "ACCEPT" to forward all packets regardless of the settings of the user interface." (https://wiki.archlinux.org/title/Uncomplicated_Firewall)
- See also: VTI on edgerouter: https://community.ui.com/questions/EdgeRouter-Routing-problem-with-IPsec-site-to-site-VPN-using-VTI-static-routing/fa1d4813-c639-4d5f-bf86-66bbdba51dee

# USG
Clients need to know route to our newly attached subnet:
- Configure USG to add custom DHCP option number 121 (https://community.ui.com/questions/Custom-DHCP-Option-33-Static-Route-in-Unifi-Controller/540f8c72-24fb-44a7-86a0-50802bec0192)
  - Networks -> DHCP Section -> Add Custom DHCP Option
  - See https://www.medo64.com/2018/01/configuring-classless-static-route-option/ for a suitable calculator

# Caveats
- The EdgeRouter will only start the connection when traffic is detected for the remote subnet

# Check connection
- ping <host on remote subnet>
