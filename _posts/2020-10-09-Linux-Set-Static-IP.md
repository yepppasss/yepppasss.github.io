---
layout: article
title: Cómo definir una dirección IP estática
tags: ubuntu centos RHEL linux IP
---

# Resumen

# Ubuntu

Ya en la versión de Ubuntu 18.04 se inicia la implementación de la herramienta Netplan, que nos permite mediante un archivo YAML, aplicar la configuración de red.
Esta herramienta sustituye al viejo archivo de configuración **/etc/network/interfaces**

# Procedimiento

Deberemos acceder a la ruta **/etc/netplan** y a continuación listaremos el archivo de configuración:
```
cd /etc/netplan; ls -l
```
El fichero de configuración puede llamarse **01-netcfg.yaml** o **50-cloud-init.yaml**.
A continuación editaremos con *VIM* o *nano* (según el que más nos guste), el archivo de configuración.
Por defecto, encontraremos una configuración del siguiente estilo:
```
network:
  version: 2
  ethernets:
    ens33:
      dhcp4: yes
```
Para establecer la dirección IP estática, procederemos a establecer la siguiente configuración (*de ejemplo*):
```
network:
  version: 2
  ethernets:
    ens33:
      dhcp4: no
      addresses: 
      - 192.168.1.222/24
      gateway4: 192.168.1.1
      nameservers:
        addresses: [8.8.8.8,1.1.1.1]
```
Una vez hecho esto, guardar y cierrar el archivo. Se aplicarán los cambios con:
```
sudo netplan apply
```
> En caso de se tenga algunos problemas, ejecutar el siguiente comando:
> ```
> sudo netplan --debug apply
> ```

# CentOS/RHEL

En la versión 7 de CentOS/RHEL, se introduce la herramienta **NetworkManager**, que permite una administración a partir de ésta herramienta.
Para las versiones previas, el método utilizado, es través de la configuración de un archivo de configuración.
A continuación vamos a ver las diversas formas de configuración:

## NetworkManager

Esta herramienta dispone de dos versiones para su administración:

* **nmcli**: versión CLI.
* **nmtui**: versión con GUI en CLI.

### Utilizando el nmcli (directamente)

Para cambiar la dirección IP de la interfaz **ens33** con un comando nmcli directo, ejecutamos:
```
sudo nmcli connection modify ens33 IPv4.address 192.168.20.2/24
```
Usaremos una sintaxis similar para cambiar la puerta de enlace y la configuración de dns:
```
sudo nmcli connection modify ens33 IPv4.gateway 192.168.20.1
sudo nmcli connection modify ens33 IPv4.dns 192.168.20.1
```
Finalmente, configuramos el método en **manual** para evitar usar cualquier otro protocolo de arranque para la interfaz. 
Este comando establece la opción *BOOTPROTO* en **none** en el archivo de configuración de la interfaz:
```
sudo nmcli connection modify ens33 IPv4.method manual
```
### Utilizando el nmcli (shell)
También disponemos de la posibilidad de realizar la configuración mediante un método interactivo, para realizar los cambios.
Para ello, ingresaremos en la shell:
```
sudo nmcli connection edit ens35
```
Nos aparecerá el siguiente mensaje de acceso al shell:
```
===| Editor de conexión interactivo de nmcli |===

Modificando la conexión «802-3-ethernet» existente: «ens35»

Escriba «help» o «?» para comandos disponibles.
Escriba «print» para mostrar todas las propiedades de conexión.
Escriba «describe [<parámetro>.<prop>]» para una descripción de propiedad detallada.

Puede modificar los siguientes parámetros: connection, 802-3-ethernet (ethernet), 802-1x, dcb, sriov, ethtool, match, ipv4, ipv6, tc, proxy
```
Para cambiar la dirección de nuestra interfaz:
```
set IPv4.address 192.168.20.2/24
```
Nos indicará si deseamos establecer el **ipv4.method** a manual y le indicaremos que si:
```
¿Desea también establecer «ipv4.method» a «manual»? [yes]: yes
```
Para cambiar las otras propiedades:
```
set IPv4.gateway 192.168.20.1
set IPv4.dns 192.168.20.1
set IPv4.method manual
```
Para guardar las modificaciones que hemos realizado, utilizaremos el siguiente comando:
```
save
```
Para salir de la Shell, utilizaremos el comando `quit`.
Para que tengan efecto los cambios, será necesario recargar la conexión que podemos realizar con el siguiente comando:
```
sudo nmcli connection down ens35 && sudo nmcli connection up ens35
```



---
