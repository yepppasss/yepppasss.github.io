---
layout: article
title: Cómo configurar el proxy en Linux
tags: linux proxy
---

# Resumen

En todas las distribuciones de Linux, la configuración del proxy se puede establecer mediante variables de entorno. 
Hay una serie de variables disponibles para usar, que van desde el tráfico HTTP hasta el tráfico FTP.

La configuración del proxy puede ser persistente configurándola en su perfil o no persistente configurándola desde la sesión de shell.

## Variables de entorno proxy

 **Variable** | **Description** 
---|---
 http_proxy | Servidor proxy para tráfico HTTP.
 https_proxy | Servidor proxy para tráfico HTTPS.
 ftp_proxy | Servidor proxy para tráfico FTP.
 no_proxy | Patrón para direcciones IP o nombres de dominio que no deberían usar el proxy.

El valor para cada configuración de proxy, excepto para no_proxy, usa la misma plantilla. 
Todos requieren un nombre de host, pero opcionalmente puede especificar un puerto de servidor proxy y sus credenciales de usuario si es necesario. 
Por ejemplo:

```
proxy_http=username:password@proxy-host:port
```
# Configuración de proxy temporal para un único usuario

Es posible que no siempre desee forzar el tráfico de Internet a través de un proxy. 
A veces es necesario anular la configuración existente, y puede hacerlo de forma segura configurando las variables de entorno del proxy desde la línea de comandos.

Lo siguiente establecerá un proxy para HTTP y HTTPS, mientras evita que el tráfico local pase por el proxy. 
Nuestro *endpoint* de servidor proxy de ejemplo es `my.proxy.server:8080` para el tráfico HTTP y `my.proxy.server:8081` para HTTPS.

* Abriremos una ventana de terminal en la que se necesite acceso de proxy.
* Estableceremos y exportaremos la variable HTTP_PROXY.
```
export HTTP_PROXY=user:pass@my.proxy.server:8080
```
* Estableceremos y exportaremos la variable HTTPS_PROXY.
```
export HTTPS_PROXY=user:pass@my.proxy.server:8081
```
* Estableceremos y exportaremos la variable NO_PROXY para evitar que se envíe tráfico local al proxy.
```
xport NO_PROXY=localhost,127.0.0.1,*.my.lan.domain
```



---
