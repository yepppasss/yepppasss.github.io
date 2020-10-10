---
layout: article
title: [Linux] Establecer comunicación SSH para sesión sin contraseña
tags: linux ssh
---

# Resumen

En la administración de sistemas es muy habitual realizar conexiones SSH entre equipos. 
Para agilizar la administración de ellos, existe una manera que permite compartir la clave SSH publica y con ello, ya no es requerida la autenticación.
En este artículo se procederá a explicar los pasos para realizarlo.

# Procedimiento

En el procedimiento utilizaremos los siguientes servidores:

* **Servidor local**: 172.16.102.137
* **Servidor remoto**: 172.16.102.138

El primer paso será acceder al **servidor local**. 
Una vez dentro del servidor, ejecutamos el siguiente comando:
```
ssh-keygen -t rsa -b 2048
```
El siguiente paso, será realizar una copia de la llave en el **servidor remoto** mediante el siguiente comando:
```
ssh-copy-id linux@172.16.102.138
```
Con ésto ya podremos conectaremos por SSH al **servidor remoto**:
```
ssh linux@172.16.102.138
```
# Verificación de claves

Una vez conectado, podemos verificar, que se ha copiado la *public key*. 
Para ello, revisaremos el contenido del archivo **.ssh/authorized_keys** dentro del servidor remoto.
```
cat .ssh/authorized_keys
```
Volvemos al servidor local y revisamos la *public key*.
```
cat .ssh/id_rsa.pub
```
Con esto nos aseguramos que dispone de la misma *key*, ya podemos conectar al servidor remoto sin uso de contraseña.

