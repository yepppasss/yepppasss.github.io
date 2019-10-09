---
title: Inicio de sesión SSH sin contraseña
tags: SSH Linux
aside:
  toc: true
---

SSH (Secure SHELL) es un protocolo de red de código abierto más confiable, que se utiliza para iniciar sesión en servidores remotos para la ejecución de comandos y programas.

En esta entrada se mostrará un ejemplo de como configurar un inicio de sesión de conexión SSH en un servidor Linux sin contraseña.

Para mostrar el ejemplo, utilizaremos el siguiente entorno:

* **Servidor local:** 192.168.2.157 (Ubuntu)
* **Servidor remoto:** 192.168.2.131 (Ubuntu)

El primer paso será acceder al **servidor local**.
Una vez dentro del servidor, ejecutamos el siguiente comando:

```bash
ssh-keygen -t rsa -b 2048
```
<img class="image image--xl" src="/assets/images/2019-10-09-login-ssh-without-password_01.png"/>

Test

[Page layout](https://tianqi.name/jekyll-TeXt-theme/samples.html#page-layout) 
