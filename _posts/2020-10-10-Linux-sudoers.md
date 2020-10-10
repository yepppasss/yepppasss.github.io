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
# Añadir el usuario al grupo de sudo
Utilizaremos el comando `usermod` para añadir el usuario al grupo **sudo**:
```
usermod -aG sudo pepe
```
# Probar el acceso sudo
Para verificar los nuevos permisos, primero se debe utilizar el comando `su` para pasar a la nueva cuenta de usuario:
```
su - pepe
```
Como nuevo usuario, verificaremos que se pueda usar `sudo` anteponiendo `sudo` al comando que desee ejecutar con privilegios de superusuario:
```
sudo <comando>
```
La primera vez que se utilice `sudo` en una sesión, se le solicitará la contraseña de la cuenta de ese usuario.

# Asignar plenos privilegios (*like a root*)
Deberemos abrir el archivo de configuración, que podemos localizar en **/etc/sudoers** o utilizar el siguiente comando:
```
visudo
```
Al ejecutar el comando, podremos ver una configuración similar a la siguiente:
```
#
# This file MUST be edited with the 'visudo' command as root.
#
# Please consider adding local content in /etc/sudoers.d/ instead of
# directly modifying this file.
#
# See the man page for details on how to write a sudoers file.
#
Defaults  env_reset
Defaults  mail_badpass
Defaults  secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
 
# Host alias specification
 
# User alias specification
 
# Cmnd alias specification
 
# User privilege specification
root  ALL=(ALL:ALL) ALL
 
# Members of the admin group may gain root privileges
%admin ALL=(ALL) ALL
 
# Allow members of group sudo to execute any command
%sudo  ALL=(ALL:ALL) ALL
 
# See sudoers(5) for more information on "#include" directives:
 
#includedir /etc/sudoers.d
```
Deberemos añadir en la última línea el usuario al que deseamos asignar los permisos.

```
pepe    ALL=(ALL:ALL) ALL
```

Es importante conocer que los permisos tienen la siguiente correspondencia: * user **ALL**=(ALL:ALL) ALL: en este se indica que la regla se aplica a cualquier anfitrión (o *host*). * user ALL=(**ALL**:ALL) ALL: "user" podrá usar comandos de cualquier usuario. * user ALL=(ALL:**ALL**) ALL: si el anterior "ALL" permitía usar comandos de usuarios, éste lo hará de grupos. * user ALL=(ALL:ALL) **ALL**: las reglas se aplican a todos los comandos.
{:.success}

# Acceso sin contraseña a sudo
Para poder acceder sin tener que introducir la contraseña, se deberá modificar la entrada anterior archivo sudo y asignar:
```
# Allow members of group sudo to execute any command
pepe  ALL=(ALL) NOPASSWD:ALL
```
> Se recomienda utilizar el editor *VIM*. Para guardar el archivo, se requerirá utilizar `:wq!`




---
