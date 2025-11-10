# Apuntes Apache2

## ¿Qué es Apache2?
**Apache2** es un **servidor web** muy utilizado en sistemas Linux.
Su función principal es servir páginas web a través de los protocolos **HTTP y HTTPS**.

### Protocolos HTTP y HTTPS

Siglas de **HyperText Transfer Protocol**, transfiere principalmente documentos HTML y multimedia.
**HTTPS** es la **versión segura** de HTTP.

#### Puertos

```
HTTP --> Puerto 80
HTTPS --> Puerto 443
```

## Instalación

Para la instalación de Apache2 es recomendable que nuestro servidor disponga de una **IP fija**, podemos configurarla desde el archivo **Netplan**:

```bash
sudo nano /etc/netplan/50-cloud-init.yaml
```

**Nota**: Es posible que el archivo también pueda llevar el nombre <code>00-installer-config.yaml</code>, pero la ubicación del archivo será siempre en <code>/etc/netplan</code>, se deberá editar el arrchivo ubicado en ese directorio.

**Ejemplo de configuración de Netplan**:

```bash
# This file is generated from information provided by the datasource.  Changes
# to it will not persist across an instance reboot.  To disable cloud-init's
# network configuration capabilities, write a file
# /etc/cloud/cloud.cfg.d/99-disable-network-config.cfg with the following:
# network: {config: disabled}
network:
    ethernets:
        enp0s3:
            dhcp4: no
            addresses:
              - 192.168.1.130/24
            routes:
              - to: default
                via: 192.168.1.1
            nameservers:
              addresses: [8.8.8.8,8.8.4.4]
    version: 2
```

<code>enp0s3</code> --> Puerto Ethernet del servidor donde se hará la configuración.

<code>dhcp4: no</code> --> **Desactivamos la asignación de IPv4 por DHCP**, de esta forma podremos configurarla manualmente.

<code>addresses</code> --> **Asignación de direcciones IP**, se puede asignar más de una.

<code>routes</code> --> **Asignación del gateway**, normalmente será la dirección IP del Router. Puedes averiguar la IP del gateway de tu red usando el comando <code>ip route | grep default</code>:

```bash
rxn@pop-os:~$ ip route | grep default
default via 192.168.1.1 dev wlp6s0 proto dhcp metric 600 
```

Nuestro gateway es <code>192.168.1.1</code>

<code>nameservers</code> --> **Asignación de servidores DNS**, se puede asignar más de uno.

Una vez hayas configurado todo aplica los cambios y comprueba tu ip, si no sale ningún error lo has configurado correctamente:

```bash
web@ubuntu-web-server:~$ sudo netplan apply
[sudo] password for web: 
web@ubuntu-web-server:~$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host noprefixroute 
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:9c:b5:06 brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.130/24 brd 192.168.1.255 scope global enp0s3
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fe9c:b506/64 scope link 
       valid_lft forever preferred_lft forever
web@ubuntu-web-server:~$ 
```

[**Documentación oficial de Cannonical**](https://netplan.io/)

Si tu distribución es **Debian** o alguna otra **no basada en Ubuntu** o que **no disponga de Netplan** entonces se deberá configurar el archivo <code>/etc/network/interfaces</code>.

**Ejemplo de configuración de Interfaces**:

```bash
# Configuración de red estática para Debian
auto enp0s3
iface enp0s3 inet static
    address 192.168.1.130
    netmask 255.255.255.0
    gateway 192.168.1.1
    dns-nameservers 8.8.8.8 8.8.4.4

```

Luego reinicia la red con:

```bash
sudo systemctl restart networking
```

[**Guia de nixCraft sobre /etc/network/interfaces**](https://www.cyberciti.biz/faq/setting-up-an-network-interfaces-file/)

Ahora sí podremos instalar Apache:

```bash
sudo apt install apache2 -y
```

Ahora si accedemos a la ip del servidor desde el navegador veremos la página por defecto de Apache, indicando que se ha instalado correctamente:

![Página por defecto de Apache](imgs/01.png)

## Configuraciones

### Archivo /etc/apache2/ports.conf

```bash
# If you just change the port or add more ports here, you will likely also
# have to change the VirtualHost statement in
# /etc/apache2/sites-enabled/000-default.conf

Listen 80

<IfModule ssl_module>
	Listen 443
</IfModule>

<IfModule mod_gnutls.c>
	Listen 443
</IfModule>
```

Sirve para configurar que puertos escuchará el servidor web y bajo que condiciones usando la etiqueta <code>IfModule</code>:

```bash
# If you just change the port or add more ports here, you will likely also
# have to change the VirtualHost statement in
# /etc/apache2/sites-enabled/000-default.conf

Listen 80
Listen 8080

<IfModule ssl_module>
        Listen 443
        Listen 8443
</IfModule>

<IfModule mod_gnutls.c>
        Listen 443
</IfModule>
```

Ahora escuchará por los puertos **80** y **8080** para **HTTP** y **443** y **8443** para **HTTPS** cuando el **módulo SSL esté activo**.