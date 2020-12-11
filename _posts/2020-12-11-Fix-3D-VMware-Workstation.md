---
layout: article
title: Corregir problema de compatibilidad 3D en VMware Workstation 16
tags: vmware fix 3d video
---

# Resumen

Este entrada documenta como corregir la problemática detectada en una instalación de VMware Workstation 16 (instalada sobre un equipo Linux), en la que se reporta el siguiente error en el arranque de una máquina virtual de Windows.

![Image](https://raw.githubusercontent.com/yepppasss/yepppasss.github.io/master/_images/2020-12-11-Fix-3D-VMware-Workstation%20-%20001.png){:.border}

# Corrección en configuración

Antes de aplicar la corrección, deberemos verificar que tenemos soporte sobre la gráfica instalada.
Para ello ejecutaremos el siguiente comando:
```
glxinfo | grep "direct"
```
Nos debería devolver el siguiente resultado.
```
direct rendering: Yes
...
```
En caso afirmativo, se puede continuar con el procedimiento.
Para ello, a continuación editaremos el siguiente archivo:
```
vim $HOME/.vmware/preferences
```
En este archivo, buscaremos si existe la siguiente línea y si no existe la añadimos:
```
mks.gl.allowBlacklistedDrivers = "TRUE"
```
Una vez establecido este parámetro, ya se aplicará en el siguiente inicio de la máquina virtual de Windows.
