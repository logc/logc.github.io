    Title: Instalar Python 2.7 y NumPy sobre RedHat 6
    Date: 2013-12-04T11:44:57Z+0100
    Tags: python

RedHat y centOS tienen la fastidiosa dependencia de su gestor de paquetes, yum,
con la versión 2.6 de Python desde tiempos inmemoriales. Lo que en tecnología
son unos seis o siete años.

Instalar Python 2.7 no es posible con los repositorios instalados por defecto,
y si tienes la mala idea de instalar un repositorio donde sí esté y meterlo
como paquete que supedita al python por defecto, te acabas de cargar yum (y la
posibilidad más sencilla de deshacer tus cambios).

La mejor manera, pese a lo que pueda parecer, es **instalar desde fuentes**,
haciendo lo que se conoce como "altinstall" (instalación alternativa). Esto
crea ejecutables llamados `python2.7` bajo `/usr/local/bin`, que **no
sobreescriben** el comando `python` normal del sistema; éste sigue apuntando a
un ejecutable bajo `/usr/bin`.

```console
$ wget http://www.python.org/ftp/python/2.7.6/Python-2.7.6.tgz
$ tar xvfz Python-2.7.6.tgz
$ cd Python-2.7.6
$ ./configure
$ make
$ sudo make altinstall
```

Además de esto, es recomendable instalar **un `easy_install` alternativo
también**, ya que, si no, todas las librerías de python se instalarán para la
2.6 y no para la 2.7.

```console
$ wget http://pypi.python.org/packages/2.7/s/setuptools/setuptools-0.6c11-py2.7.egg
$ sudo -i
# export PATH=$PATH:/usr/local/bin
# sh setuptools-0.6c11-py2.7.egg
# exit
$ sudo rm /usr/local/bin/easy_install ## remove the symlink that shadows the system easy_install
$ # You may now try the easy_install for 2.7 with e.g.:
$ sudo /usr/local/bin/easy_install-2.7 numpy
```

Este post es una combinación de [dos respuestas en Stack Overflow](http://stackoverflow.com/questions/4149361/on-linux-suse-or-redhat-how-do-i-load-python-2-7),
y mi propia experiencia con este mismo problema.
