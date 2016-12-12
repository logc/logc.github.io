    Title: Unicode encodings
    Date: 2013-12-03T13:46:19Z+0100
    Tags: python

¡Por fin he entendido cuál es la diferencia entre Unicode y UTF-8! Y lo que es
más importante, cómo se declaran correctamente `strings` en Unicode en Python.

Unicode es **la tabla de equivalencias** entre caracteres de (casi) todos los
lenguajes humanos, y un número asignado a ese caracter en concreto.

UTF-8 es **la manera de comprimir** esos números en uno o dos bytes, en lugar
de usar 4 bytes por caracter, aprovechando el hecho de que la mayoría de
caracteres habituales están en los números bajos (menores de 128).

Es decir, si el "encodificador" encuentra un número < 128, deja ese número; si
encuentra uno mayor, pero menor de X (donde X es un límite que no recuerdo),
usa **dos bytes** para expresar este número. El "decodificador" entonces sabe
que cada caracter menor de 128 está "solo", mientras que si encuentra uno
mayor, entonces debe leer también el siguiente byte para tener el número
correcto con el que ir a la tabla Unicode y obtener el caracter adecuado.

Se puede crear una cadena Unicode en Python usando una `u` delante de la
cadena literal, pero esto no siempre es posible, como cuando se lee de un
fichero. Para convertir entre distintas codificaciones existe la función
`unicode`.

La función `unicode` **no utiliza UTF-8 por defecto**, sino, absurdamente,
ASCII. Por eso *sólo es equivalente a poner una `u` delante una cadena si se
especifica que el `encoding` sea `'utf-8'`*.

```python
unicode_string = u"Años"

unicode_str = unicode(open('some.txt', 'r').read(), encoding='utf-8')
```
