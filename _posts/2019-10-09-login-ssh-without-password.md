---
title: Inicio de sesión SSH sin contraseña
tags: SSH Linux
aside:
  toc: true
---

SSH (Secure SHELL) es un protocolo de red de código abierto más confiable, que se utiliza para iniciar sesión en servidores remotos para la ejecución de comandos y programas.

En esta entrada se mostrará un ejemplo de como configurar un inicio de sesión de conexión SSH en un servidor Linux sin contraseña.

Para mostrar el ejemplo, utilizaremos el siguiente entorno:

* **Servidor local:** 192.168.2.157
* **Servidor remoto:** 192.168.2.131

El primer paso será acceder al **servidor local**.
Una vez dentro del servidor, ejecutamos el siguiente comando:

```bash
ssh-keygen -t rsa -b 2048
```

![SSH](/assets/images/2019-10-09-login-ssh-without-password_01.png)

El siguiente paso, será realizar una copia de la llave en el **servidor remoto**, mediante el siguiente comando:

```bash
ssh-copy-id mooc@192.168.2.131
```

![SSH](/assets/images/2019-10-09-login-ssh-without-password_02.png)

A continuación, conectamos por SSH al servidor remoto:

```bash
ssh mooc@192.168.2.131
```

![SSH](/assets/images/2019-10-09-login-ssh-without-password_03.png)

Una vez conectado, podemos verificar, que se ha copiado la *public key*. 
Para ello, revisaremos el contenido del archivo **.ssh/authorized_keys** dentro del servidor remoto.

```bash
cat .ssh/authorized_keys
```

![SSH](/assets/images/2019-10-09-login-ssh-without-password_04.png)

Volvemos al servidor local y revisamos la *public key*.

```bash
cat .ssh/id_rsa.pub
```

![SSH](/assets/images/2019-10-09-login-ssh-without-password_05.png)

Tras comprobar que tiene la misma key, ya podemos conectar al servidor remoto sin uso de contraseña.
