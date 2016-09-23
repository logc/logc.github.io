    Title: py2app excludes
    Date: 2013-12-02T20:58:08Z+0100
    Tags: python

Al generar una aplicación para Mac OS X con py2app, a menudo se generan
ejecutables muy pesados, por ejemplo 20 MBs para una aplicación de línea
de comandos (!).

Uno de los problemas es que py2app tiende a incluir muchos módulos de la
librería estándar de Python que en realidad no hacen falta. Se pueden
excluir específicamente incluyéndolos en la opción excludes que se pasa
a la función setup en el archivo setup.py usado para generar la
aplicación.

Con la lista siguiente, el peso baja de 22 MBs a 6 MBs. Probablemente se
puede rebajar aún más.

```python
# File: setup.py

from setuptools import setup

APP = ['path/main.py']
Plist = {}
DATA_FILES = []
OPTIONS = {'argv_emulation': True,
           #'iconfile': 'Icons/ConverterIcon.icns',
           'plist': Plist,
           'excludes': [
               'aetypes',
               'ast',
               'bz2',
               'calendar',
               'codecs',
               'collections',
               'ctypes',
               'distutils',
               'doctest',
               'email',
               'encodings',
               'functools',
               'gzip',
               'inspect',
               'itertools',
               'locale',
               'logging',
               'optparse',
               'pickle',
               'platform',
               'pprint',
               'random',
               're',
               'sets',
               'shutil',
               'socket',
               'ssl',
               'subprocess',
               'tarfile',
               'tempfile',
               'threading',
               'traceback',
               'types',
               'unittest',
               'urllib',
               'urllib2',
               'urlparse',
               'weakref',
               'xml',
               'zipfile',
           ]
           }

setup(
    app=APP,
    package_dir={'': 'src'}, # you may not need this
    data_files=DATA_FILES,
    version="0.1",
    description="<Your description>",
    author="<Your name>",
    author_email="<Your email>",
    options={'py2app': OPTIONS},
    setup_requires=['py2app'],
)
```
