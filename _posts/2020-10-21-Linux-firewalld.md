---
layout: article
title: Cómo configurar el firewalld
tags: linux firewalld
---

# Resumen

Firewalld es una solución de administración de firewall disponible para muchas distribuciones de Linux que actúa como una interfaz para el sistema de filtrado de paquetes iptables proporcionado por el kernel de Linux. 

En este artículo se quiere cubrir cómo configurar un firewall para su servidor y se mostrará los conceptos básicos para administrar el firewall con la herramienta administrativa **firewall-cmd**.

# Conceptos básicos en Firewalld

Antes de comenzar a hablar sobre cómo usar realmente la utilidad **firewall-cmd** para administrar la configuración del firewall, debemos familiarizarnos con algunos conceptos básicos que presenta la herramienta.

## Zonas

El daemon **firewalld** administra grupos de reglas usando entidades llamadas "zonas". Las zonas son básicamente conjuntos de reglas que dictan qué tráfico debe permitirse según el nivel de confianza que tiene en las redes a las que está conectada su equipo. A las interfaces de red se les asigna una zona para dictar el comportamiento que debe permitir el firewall.

Para los equipos que pueden moverse entre redes con frecuencia (como portátiles), este tipo de flexibilidad proporciona un buen método para cambiar sus reglas según su entorno. Es posible tener reglas estrictas que prohíban la mayor parte del tráfico cuando opere en una red WiFi pública, mientras que permite restricciones más relajadas cuando está conectado a su red doméstica. Para un servidor, estas zonas no son tan importantes de inmediato porque el entorno de red rara vez, o nunca, cambia.

Independientemente de lo dinámico que pueda ser su entorno de red, es útil estar familiarizado con la idea general detrás de cada una de las zonas predefinidas para firewalld. En orden de **menos confiable** a **más confiable**, las zonas predefinidas dentro de firewalld son:

* **drop**: el nivel más bajo de confianza. Todas las conexiones entrantes se eliminan sin respuesta y solo son posibles las conexiones salientes.
* **block**: imilar al anterior, pero en lugar de simplemente eliminar conexiones, las solicitudes entrantes se rechazan con un mensaje **icmp-host-prohibited** o **icmp6-adm-prohibited*.
* **public**: representa redes públicas, no confiables. No confía en otros equipos, pero puede permitir las conexiones entrantes seleccionadas caso por caso.
* **external**: redes externas en caso de que esté utilizando el firewall como su puerta de enlace. Está configurado para el enmascaramiento de NAT para que su red interna permanezca privada pero accesible.
* **internal**: el otro lado de la zona externa, utilizado para la parte interna de una puerta de enlace. Los equipos son bastante confiables y algunos servicios adicionales están disponibles.
* **dmz**: se usa para equipos ubicados en una DMZ (equipos aislados que no tendrán acceso al resto de su red). Sólo se permiten ciertas conexiones entrantes.
* **work**: utilizado para maquinas de trabajo. Confíe en la mayoría de los equipos en la red. Se podrían permitir algunos servicios más.
* **home**: en general, implica que confía en la mayoría de los otros equipos y que se aceptarán algunos servicios más.
* **trusted**: confía en todas los equipos de la red. La más abierta de las opciones disponibles y debe usarse con moderación.

Para usar el firewall, podemos crear reglas y alterar las propiedades de nuestras zonas y luego asignar nuestras interfaces de red a las zonas que sean más apropiadas.

## Reglas permanentes

En firewalld, las reglas se pueden designar como permanentes o inmediatas. 
Si se agrega o modifica una regla, de forma predeterminada, se modifica el comportamiento del firewall que se está ejecutando actualmente. 
En el próximo arranque, se revertirán las reglas anteriores, por lo que se perderán las reglas aplicadas.

La mayoría de las operaciones de `firewall-cmd` pueden tomar la marca `--permanent` para indicar que se debe apuntar al firewall no efímero. 
Esto afectará el conjunto de reglas que se vuelve a cargar al arrancar. 
Esta separación significa que puede probar las reglas en su instancia de firewall activa y luego volver a cargar si hay problemas. 
También puede usar la marca `--permanent` para crear un conjunto completo de reglas a lo largo del tiempo que se aplicarán a la vez cuando se emita el comando de recarga.

# Instalar y habilitar su firewall para que se inicie en el arranque

**firewalld** se instala de forma predeterminada en algunas distribuciones de Linux, incluidas muchas imágenes de CentOS 7. 
Sin embargo, es posible que deba instalar firewalld usted mismo:
```
sudo yum install firewalld
```
Después de instalar **firewalld**, se puede habilitar el servicio y reiniciar su equipo. 
Tenga en cuenta que habilitar firewalld hará que el servicio se inicie en el arranque. 
Es una buena práctica crear sus reglas de firewall y aprovechar la oportunidad para probarlas antes de configurar este comportamiento para evitar problemas potenciales.
```
sudo systemctl enable firewalld
sudo reboot
```
Cuando el equipor se reinicia, su firewall debe activarse, sus interfaces de red deben colocarse en las zonas que configuró (o recurrir a la zona predeterminada configurada) y las reglas asociadas con las zonas se aplicarán a las zonas asociadas a las interfaces.
Podemos verificar que el servicio se esté ejecutando y sea accesible escribiendo:
```
sudo firewall-cmd --state
```
```
running
```
Indica que nuestro firewall está funcionando con la configuración predeterminada.

# Familiarizarse con las reglas actuales del cortafuegos
Antes de comenzar a realizar modificaciones, debemos familiarizarnos con el entorno predeterminado y las reglas proporcionadas por el daemon.

## Explorando los valores predeterminados
Podemos ver qué zona está seleccionada actualmente como predeterminada escribiendo:
```
firewall-cmd --get-default-zone
```
```
public
```
Es dado ya que no le hemos dado a firewalld ningún comando para desviarse de la zona predeterminada, y ninguna de nuestras interfaces está configurada para unirse a otra zona, esa zona también será la única zona "activa" (la zona que controla el tráfico de nuestra interfaces). Podemos verificar eso escribiendo:
```
firewall-cmd --get-active-zones
```
```
public
  interfaces: ens33

```
Auí podemos ver que nuestro equipo de ejemplo tiene una interfaz de red controladas por el firewall (ens33). 
Esta tiene definidas la reglas para la zona pública.
Sin embargo, ¿cómo sabemos qué reglas están asociadas con la zona pública? 
Podemos imprimir la configuración de la zona predeterminada escribiendo:
```
firewall-cmd --get-active-zones
```
```
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: ens33
  sources: 
  services: cockpit dhcpv6-client ssh
  ports: 
  protocols: 
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules: 
```
Podemos decir por el resultado que esta zona es tanto la predeterminada como la activa y que la interfaz *ens33* está asociada con esta zona (ya sabíamos todo esto por nuestras consultas anteriores). 
Sin embargo, también podemos ver que esta zona permite las operaciones normales asociadas con un servicio cockpit, un cliente DHCP (para asignación de dirección IP) y SSH (para administración remota).

## Explorando zonas alternativas
Ahora tenemos una buena idea sobre la configuración de la zona activa y predeterminada. 
También podemos encontrar información sobre otras zonas.
Para obtener una lista de las zonas disponibles, escriba:
```
firewall-cmd --get-zones
```
```
block dmz drop external home internal public trusted work
```
Podemos ver la configuración específica asociada con una zona al incluir el parámetro `--zone=` en nuestro comando `--list-all`:
```
sudo firewall-cmd --zone=home --list-all
```
```
home
  target: default
  icmp-block-inversion: no
  interfaces: 
  sources: 
  services: cockpit dhcpv6-client mdns samba-client ssh
  ports: 
  protocols: 
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules: 
```
Puede generar todas las definiciones de zona mediante la opción `--list-all-zones`. 
Probablemente desee canalizar la salida a un buscapersonas para facilitar la visualización:
```
sudo firewall-cmd --list-all-zones | less
```
# Selección de zonas para sus interfaces
A menos que haya configurado sus interfaces de red de otra manera, cada interfaz se colocará en la zona predeterminada cuando se inicie el firewall.

## Cambiar la zona de una interfaz
Puede realizar la transición de una interfaz entre zonas durante una sesión utilizando el parámetro `--zone=` en combinación con el parámetro `--change-interface=`. 
Al igual que con todos los comandos que modifican el cortafuegos, deberá utilizar `sudo`.
Por ejemplo, podemos hacer la transición de nuestra interfaz *ens33* a la zona de "home" escribiendo esto:
```
sudo firewall-cmd --zone=home --change-interface=ens33
```
```
success
```
> **NOTA**
> Siempre que realice la transición de una interfaz a una nueva zona, tenga en cuenta que probablemente esté modificando los servicios que estarán operativos. 
> Por ejemplo, aquí nos estamos moviendo a la zona "home", que tiene SSH disponible. Esto significa que nuestra conexión no debería caer. Algunas otras zonas no tienen SSH habilitado de forma predeterminada y si su conexión se interrumpe mientras usa una de estas zonas, es posible que no pueda volver a iniciar sesión.
Podemos verificar que esto fue exitoso preguntando nuevamente por las zonas activas:
```
sudo firewall-cmd --get-active-zones
```
```
home
  interfaces: ens33
```
## Ajuste de la zona predeterminada
Si todas sus interfaces pueden manejarse mejor en una sola zona, probablemente sea más fácil seleccionar la mejor zona predeterminada y luego usarla para su configuración. 
Puede cambiar la zona predeterminada con el parámetro `--set-default-zone=`. 
Esto cambiará inmediatamente cualquier interfaz que haya recurrido al valor predeterminado a la nueva zona:
```
sudo firewall-cmd --set-default-zone=home
```
```
success
```
# Establecer reglas para sus aplicaciones
La forma básica de definir las excepciones de firewall para los servicios que desea que estén disponibles es fácil. 
Repasaremos la idea básica aquí.

## Agregar un servicio a sus zonas
El método más sencillo es agregar los servicios o puertos que necesita a las zonas que está utilizando. 
Nuevamente, puede obtener una lista de los servicios disponibles con la opción `--get-services`:
```
sudo firewall-cmd --get-services
```
```
RH-Satellite-6 amanda-client amanda-k5-client amqp amqps apcupsd audit bacula bacula-client bb bgp bitcoin bitcoin-rpc bitcoin-testnet bitcoin-testnet-rpc bittorrent-lsd ceph ceph-mon cfengine cockpit condor-collector ctdb dhcp dhcpv6 dhcpv6-client distcc dns dns-over-tls docker-registry docker-swarm dropbox-lansync elasticsearch etcd-client etcd-server finger freeipa-4 freeipa-ldap freeipa-ldaps freeipa-replication freeipa-trust ftp ganglia-client ganglia-master git grafana gre high-availability http https imap imaps ipp ipp-client ipsec irc ircs iscsi-target isns jenkins kadmin kdeconnect kerberos kibana klogin kpasswd kprop kshell ldap ldaps libvirt libvirt-tls lightning-network llmnr managesieve matrix mdns memcache minidlna mongodb mosh mountd mqtt mqtt-tls ms-wbt mssql murmur mysql nfs nfs3 nmea-0183 nrpe ntp nut openvpn ovirt-imageio ovirt-storageconsole ovirt-vmconsole plex pmcd pmproxy pmwebapi pmwebapis pop3 pop3s postgresql privoxy prometheus proxy-dhcp ptp pulseaudio puppetmaster quassel radius rdp redis redis-sentinel rpc-bind rsh rsyncd rtsp salt-master samba samba-client samba-dc sane sip sips slp smtp smtp-submission smtps snmp snmptrap spideroak-lansync spotify-sync squid ssdp ssh steam-streaming svdrp svn syncthing syncthing-gui synergy syslog syslog-tls telnet tentacle tftp tftp-client tile38 tinc tor-socks transmission-client upnp-client vdsm vnc-server wbem-http wbem-https wsman wsmans xdmcp xmpp-bosh xmpp-client xmpp-local xmpp-server zabbix-agent zabbix-server
```
> **NOTA**
> Puede obtener más detalles sobre cada uno de estos servicios mirando su archivo **.xml** asociado dentro del directorio `/usr/lib/firewalld/services`. 
> Por ejemplo, el servicio SSH se define así:
> ```
> <?xml version="1.0" encoding="utf-8"?>
> <service>
>   <short>SSH</short>
>   <description>Secure Shell (SSH) is a protocol for logging into and executing commands on remote machines. It provides secure encrypted communications. If you plan on accessing your machine remotely via SSH over a firewalled interface, enable this option. You need the openssh-server package installed for this option to be useful.</description>
>   <port protocol="tcp" port="22"/>
> </service>
> ```
> Puede habilitar un servicio para una zona usando el parámetro `--add-service=`. 
> La operación apuntará a la zona predeterminada o cualquier zona especificada por el parámetro `--zone=`. 
> De forma predeterminada, esto solo ajustará la sesión de firewall actual. 
> Puede ajustar la configuración del firewall permanente incluyendo la marca `--permanent`.

Por ejemplo, si estamos ejecutando un servidor web que sirve tráfico HTTP convencional, podemos permitir este tráfico para las interfaces en nuestra zona "publoc" para esta sesión escribiendo:
```
sudo firewall-cmd --zone=public --add-service=http
```
```
success
```
Puede omitir la `--zone=` si desea modificar la zona predeterminada. 
Podemos verificar que la operación fue exitosa usando las operaciones `--list-all` o `--list-services`:
```
sudo firewall-cmd --zone=public --list-services
```
```
cockpit dhcpv6-client http ssh
```
Una vez que haya probado que todo funciona como debería, probablemente querrá modificar las reglas de firewall permanente para que su servicio aún esté disponible después de reiniciar. 
Podemos hacer que nuestro cambio de zona "publico" sea permanente escribiendo:
```
sudo firewall-cmd --zone=public --permanent --add-service=http
```
```
success
```
Puede verificar que esto fue exitoso agregando la marca `--permanent` a la operación `--list-services`. 
Necesita usar sudo para cualquier operación permanente:
```
sudo firewall-cmd --zone=public --permanent --list-services
```
```
cockpit dhcpv6-client http ssh
```
Su zona "public" ahora permitirá el tráfico web HTTP en el puerto 80. 
Si su servidor web está configurado para usar SSL/TLS, también querrá agregar el servicio https. 
Podemos agregar eso a la sesión actual y al conjunto de reglas permanente escribiendo:
```
sudo firewall-cmd --zone=public --add-service=https
sudo firewall-cmd --zone=public --permanent --add-service=https
```
```
cockpit dhcpv6-client http ssh
```
## ¿Qué sucede si no se dispone de un servicio adecuado?
Los servicios de firewall que se incluyen con la instalación de firewalld representan muchos de los requisitos más comunes para las aplicaciones a las que quizás desee permitir el acceso. 
Sin embargo, es probable que haya situaciones en las que estos servicios no se ajusten a sus requisitos. 
En esta situación, tiene dos opciones.

### Abrir un puerto para sus zonas
La forma más fácil de agregar soporte para su aplicación específica es abrir los puertos que usa en las zonas apropiadas. 
Esto es tan fácil como especificar el puerto o rango de puertos, y el protocolo asociado para los puertos que necesita abrir. 

Por ejemplo, si nuestra aplicación se ejecuta en el puerto *5000* y usa *TCP*, podríamos agregar esto a la zona "public" para esta sesión usando el parámetro `--add-port=`. 
Los protocolos pueden ser *tcp* o *udp*:
```
sudo firewall-cmd --zone=public --add-port=5000/tcp
```
```
success
```
Podemos verificar que esto fue exitoso usando la operación `--list-ports`:
```
sudo firewall-cmd --zone=public --list-ports
```
```
5000/tcp
```
También es posible especificar un rango secuencial de puertos separando el puerto inicial y final en el rango con un guión. 
Por ejemplo, si nuestra aplicación usa los puertos UDP *4990* a *4999*, podríamos abrirlos en "public" escribiendo:
```
sudo firewall-cmd --zone=public --add-port=4990-4999/udp
```
```
success
```
Después de la prueba, es probable que deseemos agregarlos al firewall permanente. 
Puede hacerlo escribiendo:
```
sudo firewall-cmd --zone=public --permanent --add-port=5000/tcp
sudo firewall-cmd --zone=public --permanent --add-port=4990-4999/udp
sudo firewall-cmd --zone=public --permanent --list-ports
```
```
success
success
5000/tcp 4990-4999/udp
```
## Definición de un servicio
Abrir puertos para sus zonas es fácil, pero puede ser difícil hacer un seguimiento de para qué sirve cada uno. 
Si alguna vez da de baja un servicio en su servidor, es posible que tenga dificultades para recordar qué puertos que se han abierto todavía son necesarios. 
Para evitar esta situación, es posible definir un servicio. 

Los servicios son simplemente colecciones de puertos con un nombre y una descripción asociados. 
El uso de servicios es más fácil de administrar que los puertos, pero requiere un poco de trabajo inicial. 
La forma más fácil de comenzar es copiar un script existente (que se encuentra en `/usr/lib/firewalld/services`) al directorio `/etc/firewalld/services` donde el firewall busca definiciones no estándar. 
Por ejemplo, podríamos copiar la definición de servicio SSH para usarla en nuestra definición de servicio de "example" como esta. 
El nombre del archivo menos el sufijo .xml dictará el nombre del servicio dentro de la lista de servicios del firewall:
```
sudo cp /usr/lib/firewalld/services/ssh.xml /etc/firewalld/services/example.xml
```
Ahora, puede ajustar la definición que se encuentra en el archivo que copió:
```
sudo vim /etc/firewalld/services/example.xml
```
Para comenzar, el archivo contendrá la definición SSH que copió:
```
<?xml version="1.0" encoding="utf-8"?>
<service>
  <short>SSH</short>
  <description>Secure Shell (SSH) is a protocol for logging into and executing commands on remote machines. It provides secure encrypted communications. If you plan on accessing your machine remotely via SSH over a firewalled interface, enable this option. You need the openssh-server package installed for this option to be useful.</description>
  <port protocol="tcp" port="22"/>
</service>
```
La mayor parte de esta definición son en realidad metadatos. 
Querrá cambiar el nombre corto del servicio dentro de las etiquetas `<short>`. 
Este es un nombre legible por humanos para su servicio. 
También debe agregar una descripción para tener más información si alguna vez necesita auditar el servicio. 
La única configuración que necesita hacer que realmente afecte la funcionalidad del servicio probablemente será la definición de puerto donde identifica el número de puerto y el protocolo que desea abrir. 
Esto se puede especificar varias veces.

Para nuestro servicio de "example", imagine que necesitamos abrir el puerto *7777* para *TCP* y *8888* para *UDP*. 
Al ingresar al modo INSERT presionando **i**, podemos modificar la definición existente con algo como esto:
```
<?xml version="1.0" encoding="utf-8"?>
<service>
  <short>Example Service</short>
  <description>This is just an example service.  It probably shouldn't be used on a real system.</description>
  <port protocol="tcp" port="7777"/>
  <port protocol="udp" port="8888"/>
</service>
```
Presione *ESC*, luego ingrese: **x** para guardar y cerrar el archivo. 
Vuelva a cargar su firewall para acceder a su nuevo servicio:
```
sudo firewall-cmd --reload
```
Puedes ver que ahora está entre la lista de servicios disponibles:
```
firewall-cmd --get-services
```
```
RH-Satellite-6 amanda-client amanda-k5-client amqp amqps apcupsd audit bacula bacula-client bb bgp bitcoin bitcoin-rpc bitcoin-testnet bitcoin-testnet-rpc bittorrent-lsd ceph ceph-mon cfengine cockpit condor-collector ctdb dhcp dhcpv6 dhcpv6-client distcc dns dns-over-tls docker-registry docker-swarm dropbox-lansync elasticsearch etcd-client etcd-server example finger freeipa-4 freeipa-ldap freeipa-ldaps freeipa-replication freeipa-trust ftp ganglia-client ganglia-master git grafana gre high-availability http https imap imaps ipp ipp-client ipsec irc ircs iscsi-target isns jenkins kadmin kdeconnect kerberos kibana klogin kpasswd kprop kshell ldap ldaps libvirt libvirt-tls lightning-network llmnr managesieve matrix mdns memcache minidlna mongodb mosh mountd mqtt mqtt-tls ms-wbt mssql murmur mysql nfs nfs3 nmea-0183 nrpe ntp nut openvpn ovirt-imageio ovirt-storageconsole ovirt-vmconsole plex pmcd pmproxy pmwebapi pmwebapis pop3 pop3s postgresql privoxy prometheus proxy-dhcp ptp pulseaudio puppetmaster quassel radius rdp redis redis-sentinel rpc-bind rsh rsyncd rtsp salt-master samba samba-client samba-dc sane sip sips slp smtp smtp-submission smtps snmp snmptrap spideroak-lansync spotify-sync squid ssdp ssh steam-streaming svdrp svn syncthing syncthing-gui synergy syslog syslog-tls telnet tentacle tftp tftp-client tile38 tinc tor-socks transmission-client upnp-client vdsm vnc-server wbem-http wbem-https wsman wsmans xdmcp xmpp-bosh xmpp-client xmpp-local xmpp-server zabbix-agent zabbix-server
```
Ahora puede utilizar este servicio en sus zonas como lo haría normalmente.

# Creación de sus propias zonas

Si bien las zonas predefinidas probablemente serán más que suficientes para la mayoría de los usuarios, puede ser útil definir sus propias zonas que sean más descriptivas de su función. 

Por ejemplo, es posible que desee crear una zona para su servidor web, denominada "publicweb". 
Sin embargo, es posible que desee configurar otra zona para el servicio DNS que proporciona en su red privada. 
Es posible que desee una zona llamada "privateDNS" para eso. 

Al agregar una zona, debe agregarla a la configuración del firewall permanente. 
A continuación, puede volver a cargar para llevar la configuración a su sesión en ejecución. 
Por ejemplo, podríamos crear las dos zonas que discutimos anteriormente escribiendo:
```
sudo firewall-cmd --permanent --new-zone=publicweb
sudo firewall-cmd --permanent --new-zone=privateDNS
```
Puede verificar que estos están presentes en su configuración permanente escribiendo:
```
sudo firewall-cmd --permanent --get-zones
```
```
block dmz drop external home internal privateDNS public publicweb trusted work
```
Como se indicó anteriormente, estos aún no estarán disponibles en la instancia actual del firewall:
```
sudo firewall-cmd --reload
firewall-cmd --get-zones
```
```
block dmz drop external home internal privateDNS public publicweb trusted work
```
Ahora, puede comenzar a asignar los servicios y puertos adecuados a sus zonas. 
Por lo general, es una buena idea ajustar la instancia activa y luego transferir esos cambios a la configuración permanente después de la prueba. 
Por ejemplo, para la zona "publicweb", es posible que desee agregar los servicios SSH, HTTP y HTTPS:
```
sudo firewall-cmd --zone=publicweb --add-service=ssh
sudo firewall-cmd --zone=publicweb --add-service=http
sudo firewall-cmd --zone=publicweb --add-service=https
sudo firewall-cmd --zone=publicweb --list-all
```
Asimismo, podemos agregar el servicio DNS a nuestra zona "privateDNS":
```
sudo firewall-cmd --zone=privateDNS --add-service=dns
sudo firewall-cmd --zone=privateDNS --list-all
```
```
privateDNS
  target: default
  icmp-block-inversion: no
  interfaces: 
  sources: 
  services: dns
  ports: 
  protocols: 
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules: 
```
Luego podríamos cambiar nuestras interfaces a estas nuevas zonas para probarlas:
```
sudo firewall-cmd --zone=publicweb --change-interface=ens33
```
En este punto, tiene la oportunidad de probar su configuración. 
Si estos valores funcionan para usted, querrá agregar las mismas reglas a la configuración permanente. 
Puede hacerlo volviendo a aplicar las reglas con la bandera `--permanent`:
```
sudo firewall-cmd --zone=publicweb --permanent --add-service=ssh
sudo firewall-cmd --zone=publicweb --permanent --add-service=http
sudo firewall-cmd --zone=publicweb --permanent --add-service=https
sudo firewall-cmd --zone=privateDNS --permanent --add-service=dns
``` 
Después de aplicar permanentemente estas reglas, puede reiniciar su red y volver a cargar su servicio de firewall:
```
sudo systemctl restart network
sudo systemctl reload firewalld
```
```
publicweb
  interfaces: ens33
```
Y valide que los servicios apropiados estén disponibles para ambas zonas:
```
sudo firewall-cmd --zone=publicweb --list-services
```
```
http https ssh
```
```
sudo firewall-cmd --zone=privateDNS --list-services
```
```
dns
```
Con esto se habrá configurado con éxito sus propias zonas. 
Si desea que una de estas zonas sea la predeterminada para otras interfaces, recuerde configurar ese comportamiento con el parámetro `--set-default-zone=`:
```
sudo firewall-cmd --set-default-zone=publicweb
```

---
