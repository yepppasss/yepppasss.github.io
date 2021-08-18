---
layout: article
title: Solución al VMware Kernel Module Updater de VMware Workstation en Linux
tags: vmware fix kernel linux
---

# Resumen

En esta entrada se describe el procedimiento para solventar el mensaje de VMware que solicita los header.

![Image](https://raw.githubusercontent.com/yepppasss/yepppasss.github.io/master/_images/2021-08-18-VMware-Workstation-update-kernel-module%20-%20001.png){:.border}

# Instalación de linux-header

Para ello, lo primero que haremos será buscar si ya está instalado.

```
apt search linux-headers-$(uname -r)
```

En caso de no estar instalado, ejecutaremos el siguiente comando para instalarlos:

```
apt install linux-headers-$(uname -r)
```
