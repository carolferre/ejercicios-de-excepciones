# ejercicios-de-excepciones

 Errores y excepciones
Hasta ahora los mensajes de error apenas habían sido mencionados, pero si has probado los ejemplos anteriores probablemente hayas visto algunos. Hay (al menos) dos tipos diferentes de errores: errores de sintaxis y excepciones.

8.1. Errores de sintaxis
Los errores de sintaxis, también conocidos como errores de interpretación, son quizás el tipo de queja más común que tenés cuando todavía estás aprendiendo Python:

>>>
>>> while True print('Hello world')
  File "<stdin>", line 1
    while True print('Hello world')
                   ^
SyntaxError: invalid syntax
El intérprete reproduce la línea responsable del error y muestra una pequeña “flecha” que apunta al primer lugar donde se detectó el error. El error ha sido provocado (o al menos detectado) en el elemento que precede a la flecha: en el ejemplo, el error se detecta en la función print(), ya que faltan dos puntos (':') antes del mismo. Se muestran el nombre del archivo y el número de línea para que sepas dónde mirar en caso de que la entrada venga de un programa.

8.2. Excepciones
Incluso si una declaración o expresión es sintácticamente correcta, puede generar un error cuando se intenta ejecutar. Los errores detectados durante la ejecución se llaman excepciones, y no son incondicionalmente fatales: pronto aprenderás a gestionarlos en programas Python. Sin embargo, la mayoría de las excepciones no son gestionadas por el código, y resultan en mensajes de error como los mostrados aquí:

>>>
>>> 10 * (1/0)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
ZeroDivisionError: division by zero
>>> 4 + spam*3
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
NameError: name 'spam' is not defined
>>> '2' + 2
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: Can't convert 'int' object to str implicitly
La última línea de los mensajes de error indica qué ha sucedido. Hay excepciones de diferentes tipos, y el tipo se imprime como parte del mensaje: los tipos en el ejemplo son: ZeroDivisionError, NameError y TypeError. La cadena mostrada como tipo de la excepción es el nombre de la excepción predefinida que ha ocurrido. Esto es válido para todas las excepciones predefinidas del intérprete, pero no tiene por que ser así para excepciones definidas por el usuario (aunque es una convención útil). Los nombres de las excepciones estándar son identificadores incorporados al intérprete (no son palabras clave reservadas).

El resto de la línea provee información basado en el tipo de la excepción y qué la causó.

The preceding part of the error message shows the context where the exception occurred, in the form of a stack traceback. In general it contains a stack traceback listing source lines; however, it will not display lines read from standard input.

Excepciones incorporadas lista las excepciones predefinidas y sus significados.

8.3. Gestionando excepciones
Es posible escribir programas que gestionen determinadas excepciones. Véase el siguiente ejemplo, que le pide al usuario una entrada hasta que ingrese un entero válido, pero permite al usuario interrumpir el programa (usando Control-C o lo que soporte el sistema operativo); nótese que una interrupción generada por el usuario es señalizada generando la excepción KeyboardInterrupt.

>>>
>>> while True:
...     try:
...         x = int(input("Please enter a number: "))
...         break
...     except ValueError:
...         print("Oops!  That was no valid number.  Try again...")
...
La declaración try funciona de la siguiente manera:

Primero, se ejecuta la cláusula try (la(s) linea(s) entre las palabras reservadas try y la except).

Si no ocurre ninguna excepción, la cláusula except se omite y la ejecución de la cláusula try finaliza.

Si ocurre una excepción durante la ejecución de la cláusula try el resto de la cláusula se omite. Entonces, si el tipo de excepción coincide con la excepción indicada después de la except, la cláusula except se ejecuta, y la ejecución continua después de la try.

Si ocurre una excepción que no coincide con la indicada en la cláusula except se pasa a los try más externos; si no se encuentra un gestor, se genera una unhandled exception (excepción no gestionada) y la ejecución se interrumpe con un mensaje como el que se muestra arriba.

Una declaración try puede tener más de un except, para especificar gestores para distintas excepciones. A lo sumo un bloque será ejecutado. Sólo se gestionarán excepciones que ocurren en el correspondiente try, no en otros bloques del mismo try. Un except puede nombrar múltiples excepciones usando paréntesis, por ejemplo:

... except (RuntimeError, TypeError, NameError):
...     pass
Una clase en una cláusula except es compatible con una excepción si es de la misma clase o de una clase derivada de la misma (pero no de la otra manera — una cláusula except listando una clase derivada no es compatible con una clase base). Por ejemplo, el siguiente código imprimirá B, C y D, en ese orden:

class B(Exception):
    pass

class C(B):
    pass

class D(C):
    pass

for cls in [B, C, D]:
    try:
        raise cls()
    except D:
        print("D")
    except C:
        print("C")
    except B:
        print("B")
Nótese que si las cláusulas except estuvieran invertidas (con except B primero), habría impreso B, B, B — se usa la primera cláusula except coincidente.

El último except puede omitir el nombre de la excepción capturada y servir como comodín. Se debe usar esta posibilidad con extremo cuidado, ya que de esta manera es fácil ocultar un error real de programación. También puede usarse para mostrar un mensaje de error y luego re-generar la excepción (permitiéndole al que llama, gestionar también la excepción):

import sys

try:
    f = open('myfile.txt')
    s = f.readline()
    i = int(s.strip())
except OSError as err:
    print("OS error: {0}".format(err))
except ValueError:
    print("Could not convert data to an integer.")
except:
    print("Unexpected error:", sys.exc_info()[0])
    raise
Las declaraciones try … except tienen un bloque else opcional, el cual, cuando está presente, debe seguir a los except. Es útil para aquel código que debe ejecutarse si el bloque try no genera una excepción. Por ejemplo:

for arg in sys.argv[1:]:
    try:
        f = open(arg, 'r')
    except OSError:
        print('cannot open', arg)
    else:
        print(arg, 'has', len(f.readlines()), 'lines')
        f.close()
El uso de la cláusula else es mejor que agregar código adicional en la cláusula try porque evita capturar accidentalmente una excepción que no fue generada por el código que está protegido por la declaración try … except.

Cuando ocurre una excepción, puede tener un valor asociado, también conocido como el argumento de la excepción. La presencia y el tipo de argumento depende del tipo de excepción.

La cláusula except puede especificar una variable después del nombre de excepción. La variable se vincula a una instancia de la excepción con los argumentos almacenados en instance.args. Por conveniencia, la instancia de excepción define __str__() para que se pueda mostrar los argumentos directamente, sin necesidad de hacer referencia a .args. También se puede instanciar la excepción primero, antes de generarla, y agregarle los atributos que se desee:

>>>
>>> try:
...     raise Exception('spam', 'eggs')
... except Exception as inst:
...     print(type(inst))    # the exception instance
...     print(inst.args)     # arguments stored in .args
...     print(inst)          # __str__ allows args to be printed directly,
...                          # but may be overridden in exception subclasses
...     x, y = inst.args     # unpack args
...     print('x =', x)
...     print('y =', y)
...
<class 'Exception'>
('spam', 'eggs')
('spam', 'eggs')
x = spam
y = eggs
Si una excepción tiene argumentos, estos se imprimen como en la parte final (el “detalle”) del mensaje para las excepciones no gestionadas (“Unhandled exception”).

Los gestores de excepciones no se encargan solamente de las excepciones que ocurren en el bloque try, también gestionan las excepciones que ocurren dentro de las funciones que se llaman (inclusive indirectamente) dentro del bloque try. Por ejemplo:

>>>
>>> def this_fails():
...     x = 1/0
...
>>> try:
...     this_fails()
... except ZeroDivisionError as err:
...     print('Handling run-time error:', err)
...
Handling run-time error: division by zero
8.4. Lanzando excepciones
La declaración raise permite al programador forzar a que ocurra una excepción específica. Por ejemplo:

>>>
>>> raise NameError('HiThere')
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
NameError: HiThere
El único argumento de raise indica la excepción a generarse. Tiene que ser o una instancia de excepción, o una clase de excepción (una clase que hereda de Exception). Si se pasa una clase de excepción, la misma será instanciada implícitamente llamando a su constructor sin argumentos:

raise ValueError  # shorthand for 'raise ValueError()'
Si es necesario determinar si una excepción fue lanzada pero sin intención de gestionarla, una versión simplificada de la instrucción raise te permite relanzarla:

>>>
>>> try:
...     raise NameError('HiThere')
... except NameError:
...     print('An exception flew by!')
...     raise
...
An exception flew by!
Traceback (most recent call last):
  File "<stdin>", line 2, in <module>
NameError: HiThere
8.5. Exception Chaining
The raise statement allows an optional from which enables chaining exceptions. For example:

# exc must be exception instance or None.
raise RuntimeError from exc
This can be useful when you are transforming exceptions. For example:

>>>
>>> def func():
...     raise IOError
...
>>> try:
...     func()
... except IOError as exc:
...     raise RuntimeError('Failed to open database') from exc
...
Traceback (most recent call last):
  File "<stdin>", line 2, in <module>
  File "<stdin>", line 2, in func
OSError

The above exception was the direct cause of the following exception:

Traceback (most recent call last):
  File "<stdin>", line 4, in <module>
RuntimeError: Failed to open database
Exception chaining happens automatically when an exception is raised inside an except or finally section. Exception chaining can be disabled by using from None idiom:

>>>
try:
    open('database.sqlite')
except OSError:
    raise RuntimeError from None

Traceback (most recent call last):
  File "<stdin>", line 4, in <module>
RuntimeError
For more information about chaining mechanics, see Excepciones incorporadas.

8.6. Excepciones definidas por el usuario
Los programas pueden nombrar sus propias excepciones creando una nueva clase excepción (mirá Clases para más información sobre las clases de Python). Las excepciones, típicamente, deberán derivar de la clase Exception, directa o indirectamente.

Las clases de Excepción pueden ser definidas de la misma forma que cualquier otra clase, pero es habitual mantenerlas lo más simples posible, a menudo ofreciendo solo un número de atributos con información sobre el error que leerán los gestores de la excepción. Al crear un módulo que puede lanzar varios errores distintos, una práctica común es crear una clase base para excepciones definidas en ese módulo y extenderla para crear clases excepciones específicas para distintas condiciones de error:

class Error(Exception):
    """Base class for exceptions in this module."""
    pass

class InputError(Error):
    """Exception raised for errors in the input.

    Attributes:
        expression -- input expression in which the error occurred
        message -- explanation of the error
    """

    def __init__(self, expression, message):
        self.expression = expression
        self.message = message

class TransitionError(Error):
    """Raised when an operation attempts a state transition that's not
    allowed.

    Attributes:
        previous -- state at beginning of transition
        next -- attempted new state
        message -- explanation of why the specific transition is not allowed
    """

    def __init__(self, previous, next, message):
        self.previous = previous
        self.next = next
        self.message = message
La mayoría de las excepciones se definen con nombres acabados en «Error», de manera similar a la nomenclatura de las excepciones estándar.

Muchos módulos estándar definen sus propias excepciones para reportar errores que pueden ocurrir en funciones propias. Se puede encontrar más información sobre clases en el capítulo Clases.

8.7. Definiendo acciones de limpieza
La declaración try tiene otra cláusula opcional cuyo propósito es definir acciones de limpieza que serán ejecutadas bajo ciertas circunstancias. Por ejemplo:

>>>
>>> try:
...     raise KeyboardInterrupt
... finally:
...     print('Goodbye, world!')
...
Goodbye, world!
KeyboardInterrupt
Traceback (most recent call last):
  File "<stdin>", line 2, in <module>
Si una cláusula finally está presente, el bloque finally se ejecutará al final antes de que todo el bloque try se complete. La cláusula finally se ejecuta independientemente de que la cláusula try produzca o no una excepción. Los siguientes puntos explican casos más complejos en los que se produce una excepción:

Si ocurre una excepción durante la ejecución de la cláusula try, la excepción podría ser gestionada por una cláusula except. Si la excepción no es gestionada por una cláusula except, la excepción es relanzada después de que se ejecute el bloque de la cláusula finally.

Podría aparecer una excepción durante la ejecución de una cláusula except o else. De nuevo, la excepción será relanzada después de que el bloque de la cláusula finally se ejecute.

If the finally clause executes a break, continue or return statement, exceptions are not re-raised.

Si el bloque try llega a una sentencia break, continue o return, la cláusula finally se ejecutará justo antes de la ejecución de dicha sentencia.

Si una cláusula finally incluye una sentencia return, el valor retornado será el de la cláusula finally, no la del de la sentencia return de la cláusula try.

Por ejemplo:

>>>
>>> def bool_return():
...     try:
...         return True
...     finally:
...         return False
...
>>> bool_return()
False
Un ejemplo más complicado:

>>>
>>> def divide(x, y):
...     try:
...         result = x / y
...     except ZeroDivisionError:
...         print("division by zero!")
...     else:
...         print("result is", result)
...     finally:
...         print("executing finally clause")
...
>>> divide(2, 1)
result is 2.0
executing finally clause
>>> divide(2, 0)
division by zero!
executing finally clause
>>> divide("2", "1")
executing finally clause
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "<stdin>", line 3, in divide
TypeError: unsupported operand type(s) for /: 'str' and 'str'
Como se puede ver, la cláusula finally siempre se ejecuta. La excepción TypeError lanzada al dividir dos cadenas de texto no es gestionado por la cláusula except y por lo tanto es relanzada luego de que se ejecuta la cláusula finally.

En aplicaciones reales, la cláusula finally es útil para liberar recursos externos (como archivos o conexiones de red), sin importar si el uso del recurso fue exitoso.

8.8. Acciones predefinidas de limpieza
Algunos objetos definen acciones de limpieza estándar para llevar a cabo cuando el objeto ya no necesario, independientemente de que las operaciones sobre el objeto hayan sido exitosas o no. Véase el siguiente ejemplo, que intenta abrir un archivo e imprimir su contenido en la pantalla.

for line in open("myfile.txt"):
    print(line, end="")
El problema con este código es que deja el archivo abierto por un periodo de tiempo indeterminado luego de que esta parte termine de ejecutarse. Esto no es un problema en scripts simples, pero puede ser un problema en aplicaciones más grandes. La declaración with permite que los objetos como archivos sean usados de una forma que asegure que siempre se los libera rápido y en forma correcta.:

with open("myfile.txt") as f:
    for line in f:
        print(line, end="")
Una vez que la declaración se ejecuta, el fichero f siempre se cierra, incluso si aparece algún error durante el procesado de las líneas. Los objetos que, como los ficheros, posean acciones predefinidas de limpieza lo indicarán en su documentación.
