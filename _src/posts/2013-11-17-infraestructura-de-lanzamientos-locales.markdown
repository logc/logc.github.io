    Title: Infraestructura de lanzamientos locales
    Date: 2013-11-17T21:43:12Z+0100
    Tags: sysadmin

Este post describe cómo crear una infraestructura de lanzamientos locales, para
compartir paquetes locales con otros desarrolladores del mismo equipo,
basándose en Python y Macports. Esto presupone que los otros miembros del
equipo usan todos Mac OS X, y que están interesados en usar paquetes de Python,
claro.

Necesitaremos:

- crear un paquete de Python con su correspondiente `setup.py`
- tener un directorio `releases` servido por HTTP por el servidor Apache
  integrado en el sistema operativo.
- tener un directorio `ports` servido por rsync.
- configurar Macports para usar este nuevo repositorio.

<!-- more -->

## Crear un paquete de ejemplo en Python

Crea un directorio llamado `pyhello`, y los siguientes directorios y ficheros
dentro de él:

```console
.
├── pyhello
│   ├── __init__.py
│   └── main.py
└── setup.py
```

El contenido de `setup.py`:

```python
from setuptools import setup

setup(name='pyhello',
      version='0.1',
      packages=['pyhello'],
      entry_points={
          'console_scripts': ['pyhello=pyhello.main:main']})
```

El contenido de `pyhello/main.py`:

```python
def main():
    print "Hello from pyhello!"
```

Después crea una distribución de fuentes con el comando estándar:

```console
python setup.py sdist
```

Esto crea un lanzamiento llamado `pyhello-0.1.tar.gz` bajo el directorio
`dist`.

## Servir lanzamientos por HTTP

Crea un directorio `releases` bajo `~/Sites`. Copia el fichero
`dist/pyhello-0.1.tar.gz` allá.

Crea un fichero de configuración para Apache en `/etc/apache2/users/YOUR_USERNAME.conf`

El contenido de `YOUR_USERNAME.conf`:

```apacheconf
<Directory "/Users/YOUR_USERNAME/Sites/">
    Options Indexes Multiviews
    AllowOverride AuthConfig Limit
    Order allow,deny
    Allow from all
</Directory>
```

Para lanzar Apache sólo durante esta sesión de la máquina:

```console
$ sudo apachectl start
```

Para lanzar Apache automáticamente cada vez que se inicia la máquina:

```console
$ sudo launchctl load -w /System/Library/LaunchDaemons/org.apache.httpd.plist
```

## Crear un fichero Portfile

El contenido de un fichero Portfile para un proyecto de Python normal (basado
en un `setup.py`) es:

```
# -*- coding: utf-8; mode: tcl; tab-width: 4; truncate-lines: t; indent-tabs-mode: nil; c-basic-offset: 4 -*- vim:fenc=utf-8:et:sw=4:ts=4:sts=4
# $Id$

PortSystem 1.0
PortGroup python 1.0

name                py-pyhello
version             0.1

categories-append   devel
license             BSD
maintainers         your.mail.provider.com:your.mail.name
description         An example Python app
long_description    Pyhello is an example Python app, distributed over \
                    local Macports

platforms           darwin
supported_archs     noarch

homepage            http://your.local.machine/~YOUR_USERNAME/
master_sites        http://your.local.machine/~YOUR_USERNAME/releases/
distname            pyhello-${version}

python.versions     26 27

checksums           rmd160 b8a9e7e5db29d407db95310ee95e7048d6fa1cdd \
                    sha256 cd8693792f5e6632c0c7efd287b5153571173cc3a26f98c2cead7af6a0445b2b
```

**Aclaraciones**:

- `PortGroup` para python configura los comandos de `build` e `install`;
  también existen grupos para `haskell` y otros.

- el nombre del paquete es manipulado para generar un paquete por cada versión
  listada en `python.versions`. En este ejemplo, se generarían `py26-pyhello` y
  `py27-pyhello`.

- la convención para el campo `maintainers` es `gmail.com:user` para
  `user@gmail.com`

- para generar los digests del lanzamiento con los siguientes comandos:

```console
$ openssl sha256 dist/pyhello-0.1.tar.gz
$ openssl rmd160 dist/pyhello-0.1.tar.gz
```

## Servir Portfiles por rsync

Crea un directorio `ports` bajo `~/Sites`, y los siguentes subdirectorios en él:

```console
.
└── python
    └── pyhello
        └── Portfile
```

Copia el fichero Portfile a su ubicación mostrada y ejecuta el siguiente comando en el directorio `ports`:

```console
$ portindex
Creating port index in ...
Adding port python/pyhello
Adding subport py26-pyhello
Adding subport py27-pyhello

Total number of ports parsed:	3
Ports successfully parsed:	3
Ports failed:			0
Up-to-date ports skipped:	0
```

Aunque rsync viene instalado por defecto, usaremos la versión instalada con
Macports en lo siguiente. La versión por defecto está en `/usr/bin`.

Crear un fichero para `launchctl` en
`/Library/LaunchDaemons/name.YOUR_USERNAME.rsync.plist`

El contenido de `rsync.plit`:

```xml
 <?xml version="1.0" encoding="UTF-8"?>
 <!DOCTYPE plist PUBLIC "-//Apple Computer//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
 <plist version="1.0">
 <dict>
        <key>Disabled</key>
        <false/>
        <key>Label</key>
        <string>name.YOUR_USERNAME.rsync</string>
        <key>Program</key>
        <string>/opt/local/bin/rsync</string>
        <key>ProgramArguments</key>
        <array>
                <string>/opt/local/bin/rsync</string>
                <string>--daemon</string>
                <string>--config=/opt/local/etc/rsyncd.conf</string>
        </array>
        <key>inetdCompatibility</key>
        <dict>
                <key>Wait</key>
                <false/>
        </dict>
        <key>Sockets</key>
        <dict>
            <key>Listeners</key>
            <dict>
                <key>SockServiceName</key>
                <string>rsync</string>
                <key>SockType</key>
                <string>stream</string>
            </dict>
        </dict>
 </dict>
 </plist>
```

El contenido de `/opt/local/etc/rsyncd.conf`:

```ini
[ports]

path=/Users/YOUR_USERNAME/Sites/ports
log file=/opt/local/var/log/rsyncd.log
pid file=/opt/local/var/run/rsyncd.pid
read only=true
```

Añadir rsync a `launchctl`:

```console
$ sudo launchctl load -w /Library/LaunchDaemons/name.YOUR_USERNAME.rsync.plist
```

Comprueba que sólo se pueden listar los ports y nada más del sistema de
fichero:

```console
$ rsync -axv rsync://localhost/
$ rsync -axv rsync://localhost/ports/
```

## Configurar Macports

Configura Macports en todas las máquinas que deban poder acceder a los nuevos
paquetes. El nuevo repositorio debe ir antes del repositorio por defecto para
evitar conflictos de nombres.

```
rsync://localhost/ports/
rsync://rsync.macports.org/release/ports/ [default]
```

Comprueba desde otras máquinas que se pueden encontrar los nuevos paquetes:

```console
$ port search pyhello
py-pyhello @0.1 (python, devel)
    An example Python app

py26-pyhello @0.1 (python, devel)
    An example Python app

py27-pyhello @0.1 (python, devel)
    An example Python app
```

Después de instalar el nuevo paquete, sus comandos están a nuestra disposición:

```console
$ sudo port install py27-pyhello
$ pyhello
Hello from pyhello!
```
