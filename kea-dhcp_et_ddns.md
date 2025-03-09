### Mise en place d'un Server DHCP avec KEA :

1- On met à jour les repo :

```bash
apt update
```

2- On installe **kea-dhcp4-server** :

```bash
apt install kea-dhcp4-server -y
```

3- On configure le DHCP dans le fichier : **kea-dhcp4.conf** dans le dossier _/etc/kea/_ :

```bash
{
  "Dhcp4": {
    "interfaces-config": {
      "interfaces": [ "enp0s3" ],
    "dhcp-socket-type": "raw"
    },

"dhcp-ddns" : {
      "enable-updates": true
    },
     "ddns-qualifying-suffix": "dhcp.labo",
     "ddns-override-client-update": true,
     "ddns-update-on-renew": true,


    "subnet4": [
      {
        "subnet": "192.168.100.0/24",
        "pools": [
          { "pool": "192.168.100.10 - 192.168.100.150" }
        ],
        "option-data": [
          {
            "name": "routers",
            "data": "192.168.100.254"
          },
         {
            "name": "domain-name",
            "data": "dhcp.labo"
          },
          {
            "name": "domain-name-servers",
            "data": "192.168.100.251, 8.8.8.8"
          }
        ]
      },

      {
        "subnet": "192.168.200.0/24",
        "pools": [
          { "pool": "192.168.200.10 - 192.168.200.50" }
        ],
        "option-data": [
          {
            "name": "routers",
            "data": "192.168.200.254"
          },
          {
            "name": "domain-name-servers",
            "data": "192.168.100.251, 8.8.8.8"
          }
        ]
      }
    ]
  }
}


```

4- Vérification / Logs

- Pour vérifier la validité du format JSON de la config kea-dhcp.conf :

```bash
apt install jsonlint
jsonlint-php /etc/kea/kea-dhcp4.conf
```

- Pour vérifier les logs / journaux :

```bash
journalctl -u kea-dhcp4-server.service -e
```

5- On redémarre le serveur DHCP :

```bash
systemctl restart kea-dhcp4-server.service
```

### DHCP sur le Client :

1- Flush IP sur une interface : > ip a flush \<interface>

```bash
ip a flush enp0s3
```

2-

- Arrêter le processus dhcp avec :

```bash
dhclient -r
```

- Regénérer une requête dhcp :

```bash
dhclient -v
```

### Mise en place d'un Serveur DNS avec Bind9 :

1- On met à jour les repo :

```bash
apt update
```

2- On installe **bind9** :

```bash
apt install bind9 bind9-utils -y
```

3- On configure le DHCP dans le fichier : **named.conf** ; **named.conf.options** et **named.conf.local** dans le dossier _/etc/bind/_ :

a) Dans le fichier **named.conf.options** , on décommente "forwarders" et on attribue des nouveaux :

```bash
forwarders {
	1.1.1.1;
	1.0.0.1;
	8.8.8.8;
};
```

b) Dans le fichier **named.conf.local** :

```bash
zone "dhcp.labo" {
	type primary;
	file "dhcp.zone";
};
```

4- On crée le fichier de zone **dhcp.zone** dans le dossier _/var/cache/bind/_ :

a) On copie le model **_/etc/bind/db.empty_** dans **_/var/cache/bind/dhcp.zone_** :

```bash
cp /etc/bind/db.empty /var/cache/bind/dhcp.conf
```

b) On modifie la copie :

```bash
$TTL 86400
@ IN SOA SRVDHCP.dhcp.labo. root.localhost. (
											1 ; Serial
											604800 ; Refresh
											86400 ; Retry
											2419200 ; Expire
											86400 ) ; Negative Cache TTL
;
@  IN   NS  SRVDHCP.dhcp.labo.
        A   192.168.100.250
SRVDHCP A   192.168.100.250
www     CNAME SRVDHCP
R-0     A   192.168.100.254
```

5- On redémarre le serveur DNS :

```bash
systemctl restart named.service
```

6- Vérification / Logs :

- Pour vérifier la validité du format du fichier de zone **dhcp.zone** :

```bash
named-checkconf -z
```

- Pour vérifier les logs / journaux :

```bash
journalctl -u named.service -e
```
