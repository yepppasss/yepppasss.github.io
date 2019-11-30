---
title: Como encontrar la edición y versión de Windows
tags: Windows version
aside:
  toc: true
---

En ocasiones nos podemos encontrar con ISOS de Windows, de las que desconocemos la versión que instala. En esta entrada, se explicará como poder obtener la versión de Windows a través de la ISO.

Para obtener dicha información, seguiremos los siguientes pasos:

1. Montar la imagen ISO en un sistema Windows.
2. Una vez montada la imagen, accedemos a la carpeta **sources**.
3. Dentro, buscaremos un archivo **install.wim**. Nos podemos encontrar que no exista este fichero y tengamos algunos de los siguientes **install.swm** o **install.esd**
4. Una vez localizado, ejecutaremos el siguiente comando desde el **símbolo del sistema** o desde **powershell**:
 * Para **install.wim**: dism /Get-WimInfo /WimFile:D:\sources\install.wim /index:1
 * Para **install.swm**: dism /Get-WimInfo /WimFile:D:\sources\install.swm /index:1
 * Para **install.esd**: dism /Get-WimInfo /WimFile:D:\sources\install.esd /index:1

Aparecerá una salida como la que se muestra en la siguiente captura. 

![SSH](/assets/images/2019-10-30-how-to-find-the-edition-and-version-of-windows_01.png)
