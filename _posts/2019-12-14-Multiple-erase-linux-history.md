---
title: Borrado de múltiples registros de history, sin dejar rastro
tags: Linux history
aside:
  toc: true
---

Para poder realizar un borrado de un conjunto de línea, podemos hacer uso de un *script* que permite realizar el borrado de múltiples líneas de registro.
Veamos el siguiente ejemplo de **history**:

```bash
  174  sudo -i
  175  jobs
  176  sudo -i
  177  sudo -i
  178  sudo i-
  179  sudo -i
  180  history
```
Si deseamos borrar todos los registros después del 175, podemos hacer uso del siguiente *script*

```bash
for i in {1..5}; do history -d 176; done
```

Con esto borramos el rango especificado (5 registros desde el 175), pero este comando que hemos ejecutado, se registrará también en el **history**.
Para ello añadiremos la siguiente instrucción al comando anterior:

```bash
history -d $(history 1)
``` 
Quedando:

```bash
for i in {1..5}; do history -d 176; done; history -d $(history 1)
```

Con este comando, se completará el borrado de los registros indicados y no se registrará en el **history** la ejecución de éste.
