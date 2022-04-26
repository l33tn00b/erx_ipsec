# erx_ipsec
IPSec connection ER-X to Ubuntu host

# Edgerouter
Settings:  
![Settings EdgeRouter](https://github.com/l33tn00b/erx_ipsec/blob/main/settings_er.png)

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
- since this is a bogus net on out server, add an appropriate ip to the loopback interface
  `ip addr add 10.8.5.1/24 dev lo `
- add route: `ip route add 192.168.3.0/24 via 10.8.5.1`

# Caveats
- The EdgeRouter will only start the connection when traffic is detected for the remote subnet

# Check connection
- ping <host on remote subnet>
