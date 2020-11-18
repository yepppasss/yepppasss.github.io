---
layout: article
title: Cómo instalar PowerCLI en CentOS 8
tags: linux centos powercli
---

# Resumen

La herramienta VMware PowerCLI es un conjunto de módulos de Powershell que se utilizan para gestionar y mantener un entorno virtual de VMware.
Tras la llegada de Powershell a linux, se han abierto numerosas nuevas opciones de administración de sistemas desde un entorno Linux, como es el caso de VMware PowerCLI.
En este artículo, se procede a describir los pasos requeridos para poder realizar la instalación de Powershell + PowerCLI en un servidor CentOS.

## Instalación de Powershell

En este punto se añadirá la entrada de repositorio y su posterior instalación.

```
curl https://packages.microsoft.com/config/rhel/7/prod.repo -o- | sudo tee /etc/yum.repos.d/microsoft.repo
```

A continuación procedemos a instalar Powershell:

```
sudo yum install -y powershell
```

## Instalación de PowerCLI

El siguiente paso será instalar PowerCLI, para ello accederemos a la consola de Powershell:

```
pwsh
```

A continuación procederemos a configurar y buscar el módulo de **VMware.PowerCLI**:

```
Set-PSRepository -Name "PSGallery" -InstallationPolicy "Trusted"
Find-Module "VMware.PowerCLI" | Install-Module -Scope "CurrentUser" -AllowClobber
Import-Module "VMware.PowerCLI"
```

Podemos listar los modulos que disponemos:

```
Get-Module "VMware.*" -ListAvailable | FT -Autosize
```
Generalmente para las instancias de VMware, es usual utilizar certificados SSL autofirmados. Para evitar obtener un error por ello, podremos utilizar el comando que lista a continuación
{:.warning}

```
Set-PowerCLIConfiguration -InvalidCertificateAction:Ignore
```


