---
layout: article
title: Ubuntu 18.04 - Cómo definir una dirección IP estática
tags: ubuntu 18.04 IP
---

# Resumen

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


---
