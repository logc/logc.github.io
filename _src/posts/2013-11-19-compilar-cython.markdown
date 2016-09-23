    Title: Compilar Cython
    Date: 2013-11-19T08:30:14Z+0100
    Tags: python

Compilar ficheros de Cython es más fácil usando la función `cythonize`
dentro del setup.py de nuestro proyecto:

```python
# File: setup.py

from distutils.core import setup
from Cython.Build import cythonize

setup(
      name = "My hello app",
      ext_modules = cythonize('fibonacci.pyx'), # accepts a glob pattern
)
```

Por ejemplo, el fichero a compilar es fibonacci.pyx, que contiene un
ejemplo del tutorial de Cython:

```python
# File: fibonacci.pyx

def fib(n):
    a, b = 0, 1
    while b < n:
        print b,
        a, b = b, a + b
```

El resultado de compilar `fibonacci.pyx` es un fichero con código
intermedio en C `fibonacci.c` y un fichero compilado `fibonacci.so`.

```bash
$ python setup.py build
running build
running build_ext
building 'fibonacci' extension
/usr/bin/clang ...snip... -o build/lib.macosx-10.8-x86_64-2.7/fibonacci.so
```

> **note**
>
> El compilador que se utiliza es el que sea por defecto del sistema en
> ese momento. Aún estoy investigando cómo seleccionar un compilador en
> concreto, o añadir librerías y flags.

El fichero .so puede usarse desde otros módulos de Python, o
directamente desde la shell

```pycon
>>> import fibonacci
>>> fibonacci.fib(5)
1 1 2 3
```
