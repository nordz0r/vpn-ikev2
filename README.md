## VPN IKEv2 Server with Let`s Encrypt certificates
Tested on Debian 10/11

## Requirements
### Open Firewall ports: 
* `UDP/500` `UDP/4500` (StrongSWAN)
* `TCP/80` (certbot)

### Domain name 
* Pesonal domain like vpn.domain.com

### Install Packages
```
sudo apt update 
sudo apt install -y strongswan strongswan-pki libcharon-extra-plugins python3-certbot-apache
```


### Variables 
* $DOMAIN is your domain name
* $USER is your username
* $PASSWORD is your password

### Get certificate for $DOMAIN
```
certbot certonly --apache -d $DOMAIN --register-unsafely-without-email --agree-tos
```

### Configs

* /etc/ipsec.conf
```
config setup
    charondebug = "ike 1, knl 1, cfg 1, net 1"
    uniqueids = never

conn $DOMAIN
    auto = add
    compress = no
    type = tunnel
    keyexchange = ikev2
    fragmentation = yes
    forceencaps = yes
    dpdaction = clear
    dpddelay = 300s
    rekey = no
    left = %defaultroute
    leftid = @$DOMAIN
    leftcert = /etc/letsencrypt/live/$DOMAIN/fullchain.pem
    leftsendcert = always
    leftsubnet = 0.0.0.0/0
    right = %any
    rightid = %any
    rightauth = eap-mschapv2
    rightsourceip = 10.10.10.0/24
    rightdns = 1.1.1.1
    rightsendcert = never
    eap_identity = %identity
    ike = aes256-sha256-modp2048,aes128-sha1-modp1024,3des-sha1-modp1024!
    esp = aes256-sha256,aes128-sha1,3des-sha1!
    dpdaction = restart 
```

* /etc/ipsec.secrets
```
$DOMAIN : RSA "/etc/letsencrypt/live/$DOMAIN/privkey.pem"
$USER : EAP "$PASSOWRD"
```

### Configure permissions strongswan to certificates
```
sudo apt install -y apparmor-utils
sudo aa-complain /usr/lib/ipsec/charon
sudo aa-complain /usr/lib/ipsec/stroke
cat /etc/letsencrypt/live/$DOMAIN/chain.pem > /etc/ipsec.d/cacerts/ca.pem
```

### Start
```
sudo ipsec restart
sudo update-rc.d ipsec defaults
```

## TODO
- [ ] Create an executable script for automatic installation
