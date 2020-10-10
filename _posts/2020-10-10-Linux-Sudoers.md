---
layout: article
title: Establecer a un usuario sudores con plenos privilegios
tags: linux sudoer full
---

# Resumen

El comando sudo ofrece un mecanismo para conceder privilegios de administrador; normalmente solo está disponible para el usuario root para usuarios normales.
Vamos a ver en este artículo como definir la configuración para dar a un usuario plenos privilegios a través de sudo.

# Procedimiento

El primer paso será conectar con el usuario **root** al servidor que deseamos realizar las modificaciones.

# Añadir un usuario

> En el caso de disponer ya del usuario que deseamos utilizar, se puede omitir este paso.

Utilizaremos el comando `adduser` para añadir un nuevo usuario:
```
adduser pepe
```
Nos solicitará que se cree y verifique una contraseña:
```
Enter new UNIX password:
Retype new UNIX password:
passwd: password updated successfully
```
A continuación, se le solicitará que rellene la información sobre el nuevo usuario. 
Servirá aceptar los valores predeterminados y dejar toda esta información en blanco:
```
Changing the user information for sammy
Enter the new value, or press ENTER for the default
    Full Name []:
    Room Number []:
    Work Phone []:
    Home Phone []:
    Other []:
Is the information correct? [Y/n]
```

---
