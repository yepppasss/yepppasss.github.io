---
layout: article
title: Cómo desconectar a un usuario de un servidor remoto Windows
tags: windows logoff psexec pstools
---

# Resumen

En ocasiones nos encontramos en situaciones en las que intentamos acceder a un servidor remoto Windows, pero no hay sesiones disponibles por que no se han cerrado y permanecen abiertas.

Para ello existe una suite de herramientas de Microsoft llamada [PsTools](https://docs.microsoft.com/en-us/sysinternals/downloads/pstools), la cual contiene múltiples herramientas que permite la administración remota de un servidor.

En nuestro caso nos interesaremos por una en concreto **PsExec**, la cual nos permtite ejecutar procesos de forma remota.
Por ello, lo primero que haremos se realizar la descarga de la suite y almacenaremos los archivos en **C:\Windows\System32**, para que sea visible en todos los paths. 

# Procedimiento de desconexión

Una vez disponible el **psexec.exe**, procederemos a conectar al servidor deseado a través del siguiente comando:

`psexec.exe \\<nombre-servidor> -u <username> -p <password> cmd`

Donde:

- **nombre-servidor**: servidor al que intentamos acceder (puede ser por IP, nombre NETBIOS o nombre FQDN).
- **username**: usuario con derechos de administrador que utilizará para acceder al servidor de forma remota.
- **password**: contraseña del usuario.
- **cmd**: es el nombre de la aplicación remota que utilizaremos. 

Una vez accedido al servidor, ejecutaremos el siguiente comando para obtener las sesiones abiertas:

`query session`
```
 NOMBRE DE SESIαN  NOMBRE DE USUARIO        ID  ESTADO    TIPO   DISPOSITIVO
>services                                    0  Desc
 console           Administrador             1  Activo
                                             2  Desc
```
La ID de la sesión que deseamos cerrar la sesión es 2. 

Para hacer eso, ejecutamos este comando:

`logoff 2`

Con esto ya hemos cerrado la sesión activa deseada.

`query session`
```
 NOMBRE DE SESIαN  NOMBRE DE USUARIO        ID  ESTADO    TIPO   DISPOSITIVO
>services                                    0  Desc
 console           Administrador             1  Activo
```
