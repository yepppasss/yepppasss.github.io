---
layout: article
title: Como corregir “Cannot find a valid baseurl for repo” en CentOS
tags: RHEL centos repositorio linux IP
---

Uno de los errores más comunes que encuentran los usuarios de CentOS cuando usan el administrador de paquetes *YUM* 
(por ejemplo, ejecutando el comando de actualización yum), 
especialmente en un sistema recién instalado, es **“Cannot find a valid baseurl for repo: base/7/x86_64”**

En este breve artículo, mostraremos cómo solucionar el error.

# Asegurar que el sistema está conectado correctamente a Internet

Para ello, realizaremos un ping por ejemplo a google.com:

```
ping 8.8.8.8
```

Si no nos da respuesta, indica un problema de DNS o no hay conexión a Internet.
Mostramos la configuración IP:

```
ip a
```

``` 
: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether XX:XX:XX:XX:XX:XX brd ff:ff:ff:ff:ff:ff
    inet 192.168.20.4/24 brd 192.168.20.255 scope global noprefixroute ens33
       valid_lft forever preferred_lft forever
```

Deberemos configurar correctamente los DNS.
Lo podemos realizar a traves de los siguientes métodos:

## Modificar configuración de red

Modificando el archivo de configuración de red.

```
vi /etc/sysconfig/network-scripts/ifcfg-ens33
```

Modificaremos el campo de DNS:

```
DNS1=10.0.2.2 
DNS2=8.8.8.8
```
## Cambiar el resolv.conf

En caso de tener funcionando el NetworkManager, lo realizaremos a través de éste, indicando en al interfaz de red los valores de DNS del punto anterior.
En caso de no disponer el NetworkManager, podemos añadir en el archivo **/etc/resolv.conf** lo siguiente:

```
search localdomain
nameserver 192.168.175.2
nameserver 8.8.8.8
```

---
