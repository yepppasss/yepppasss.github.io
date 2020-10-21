---
layout: article
title: Cómo configurar el firewalld
tags: linux firewalld
---

# Resumen

Firewalld es una solución de administración de firewall disponible para muchas distribuciones de Linux que actúa como una interfaz para el sistema de filtrado de paquetes iptables proporcionado por el kernel de Linux. 

En este artículo se quiere cubrir cómo configurar un firewall para su servidor y se mostrará los conceptos básicos para administrar el firewall con la herramienta administrativa **firewall-cmd**.

# Conceptos básicos en Firewalld

Antes de comenzar a hablar sobre cómo usar realmente la utilidad **firewall-cmd** para administrar la configuración del firewall, debemos familiarizarnos con algunos conceptos básicos que presenta la herramienta.

## Zonas

El daemon **firewalld** administra grupos de reglas usando entidades llamadas "zonas". Las zonas son básicamente conjuntos de reglas que dictan qué tráfico debe permitirse según el nivel de confianza que tiene en las redes a las que está conectada su equipo. A las interfaces de red se les asigna una zona para dictar el comportamiento que debe permitir el firewall.

Para los equipos que pueden moverse entre redes con frecuencia (como portátiles), este tipo de flexibilidad proporciona un buen método para cambiar sus reglas según su entorno. Es posible tener reglas estrictas que prohíban la mayor parte del tráfico cuando opere en una red WiFi pública, mientras que permite restricciones más relajadas cuando está conectado a su red doméstica. Para un servidor, estas zonas no son tan importantes de inmediato porque el entorno de red rara vez, o nunca, cambia.

Independientemente de lo dinámico que pueda ser su entorno de red, es útil estar familiarizado con la idea general detrás de cada una de las zonas predefinidas para firewalld. En orden de **menos confiable** a **más confiable**, las zonas predefinidas dentro de firewalld son:

* **drop**: el nivel más bajo de confianza. Todas las conexiones entrantes se eliminan sin respuesta y solo son posibles las conexiones salientes.
* **block**: imilar al anterior, pero en lugar de simplemente eliminar conexiones, las solicitudes entrantes se rechazan con un mensaje **icmp-host-prohibited** o **icmp6-adm-prohibited*.
* **public**: representa redes públicas, no confiables. No confía en otros equipos, pero puede permitir las conexiones entrantes seleccionadas caso por caso.
* **external**: redes externas en caso de que esté utilizando el firewall como su puerta de enlace. Está configurado para el enmascaramiento de NAT para que su red interna permanezca privada pero accesible.
* **internal**: el otro lado de la zona externa, utilizado para la parte interna de una puerta de enlace. Los equipos son bastante confiables y algunos servicios adicionales están disponibles.
* **dmz**: se usa para equipos ubicados en una DMZ (equipos aislados que no tendrán acceso al resto de su red). Sólo se permiten ciertas conexiones entrantes.
* **work**: utilizado para maquinas de trabajo. Confíe en la mayoría de los equipos en la red. Se podrían permitir algunos servicios más.
* **home**: en general, implica que confía en la mayoría de los otros equipos y que se aceptarán algunos servicios más.
* **trusted**: confía en todas los equipos de la red. La más abierta de las opciones disponibles y debe usarse con moderación.

Para usar el firewall, podemos crear reglas y alterar las propiedades de nuestras zonas y luego asignar nuestras interfaces de red a las zonas que sean más apropiadas.

## Reglas permanentes

En firewalld, las reglas se pueden designar como permanentes o inmediatas. 
Si se agrega o modifica una regla, de forma predeterminada, se modifica el comportamiento del firewall que se está ejecutando actualmente. 
En el próximo arranque, se revertirán las reglas anteriores, por lo que se perderán las reglas aplicadas.

La mayoría de las operaciones de **firewall-cmd** pueden tomar la marca **--permanent** para indicar que se debe apuntar al firewall no efímero. 
Esto afectará el conjunto de reglas que se vuelve a cargar al arrancar. 
Esta separación significa que puede probar las reglas en su instancia de firewall activa y luego volver a cargar si hay problemas. 
También puede usar la marca **--permanent** para crear un conjunto completo de reglas a lo largo del tiempo que se aplicarán a la vez cuando se emita el comando de recarga.





---
