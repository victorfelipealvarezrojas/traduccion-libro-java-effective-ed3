# Capítulo 1. Introducción

Este libro está diseñado para ayudarte a hacer un uso efectivo del lenguaje de programación Java y sus bibliotecas fundamentales: java.lang, java.util y java.io, así como subpaquetes como java.util.concurrent y java.util.function. Otras bibliotecas se discuten de vez en cuando.

Este libro consta de noventa elementos, cada uno de los cuales transmite una regla. Las reglas capturan prácticas generalmente consideradas beneficiosas por los programadores más experimentados y mejores. Los elementos están agrupados libremente en once capítulos, cada uno cubriendo un aspecto amplio del diseño de software. El libro no está destinado a ser leído de principio a fin: cada elemento se sostiene por sí mismo, más o menos. Los elementos están muy interrelacionados, por lo que puedes trazar fácilmente tu propio camino a través del libro.

Se han añadido muchas características nuevas a la plataforma desde la última edición de este libro. La mayoría de los elementos en este libro utilizan estas características de alguna manera. Esta tabla muestra dónde ir para la cobertura principal de las características clave:

| Característica | Elementos |
|----------------|-----------|
| Lambdas | Elementos 42–44 |
| Streams | Elementos 45–48 |
| Optionals | Elemento 55 |
| Métodos por defecto en interfaces | Elemento 21 |
| try-with-resources | Elemento 9 |
| @SafeVarargs | Elemento 32 |
| Módulos | Elemento 15 |

La mayoría de los elementos están ilustrados con ejemplos de programas. Una característica clave de este libro es que contiene ejemplos de código que ilustran muchos patrones de diseño e idiomas. Cuando es apropiado, están referenciados al trabajo de referencia estándar en esta área [Gamma95].

Muchos elementos contienen uno o más ejemplos de programas que ilustran alguna práctica a evitar. Tales ejemplos, a veces conocidos como antipatrones, están claramente etiquetados con un comentario como `// Nunca hagas esto!`. En cada caso, el elemento explica por qué el ejemplo es malo y sugiere un enfoque alternativo.

Este libro no es para principiantes: asume que ya te sientes cómodo con Java. Si no es así, considera uno de los muchos excelentes textos introductorios, como Java Precisely de Peter Sestoft [Sestoft16]. Aunque Effective Java está diseñado para ser accesible para cualquiera con un conocimiento práctico del lenguaje, debería proporcionar material para reflexionar incluso a programadores avanzados.

La mayoría de las reglas en este libro derivan de unos pocos principios fundamentales. La claridad y la simplicidad son de suma importancia. El usuario de un componente nunca debería sorprenderse por su comportamiento. Los componentes deben ser tan pequeños como sea posible, pero no más pequeños. (Como se usa en este libro, el término componente se refiere a cualquier elemento de software reutilizable, desde un método individual hasta un marco complejo que consta de múltiples paquetes.) El código debe ser reutilizado en lugar de copiado. Las dependencias entre componentes deben mantenerse al mínimo. Los errores deben detectarse lo antes posible después de que se cometen, idealmente en tiempo de compilación.

Aunque las reglas de este libro no se aplican al 100% del tiempo, caracterizan las mejores prácticas de programación en la gran mayoría de los casos. No debes seguir estas reglas servilmente, sino violarlas solo ocasionalmente y con buena razón. Aprender el arte de la programación, como la mayoría de las otras disciplinas, consiste en primero aprender las reglas y luego aprender cuándo romperlas.

En su mayor parte, este libro no trata sobre rendimiento. Se trata de escribir programas que sean claros, correctos, utilizables, robustos, flexibles y mantenibles. Si puedes hacer eso, generalmente es un asunto relativamente simple obtener el rendimiento que necesitas (Elemento 67). Algunos elementos sí discuten preocupaciones de rendimiento, y unos pocos de estos elementos proporcionan números de rendimiento. Estos números, que se introducen con la frase "En mi máquina", deben considerarse aproximados en el mejor de los casos.

Por lo que vale, mi máquina es una Intel Core i7-4770K de cuatro núcleos de 3.5GHz ensamblada en casa, con 16 gigabytes de RAM DDR3-1866 CL9, ejecutando la versión 9.0.0.15 de OpenJDK de Azul's Zulu, sobre Microsoft Windows 7 Professional SP1 (64 bits).

Al discutir características del lenguaje de programación Java y sus bibliotecas, a veces es necesario referirse a versiones específicas. Por conveniencia, este libro usa apodos en preferencia a los nombres oficiales de las versiones. Esta tabla muestra la correspondencia entre los nombres de las versiones y los apodos:

| Nombre oficial de la versión | Apodo |
|------------------------------|-------|
| JDK 1.0.x | Java 1.0 |
| JDK 1.1.x | Java 1.1 |
| Java 2 Platform, Standard Edition, v1.2 | Java 2 |
| Java 2 Platform, Standard Edition, v1.3 | Java 3 |
| Java 2 Platform, Standard Edition, v1.4 | Java 4 |
| Java 2 Platform, Standard Edition, v5.0 | Java 5 |
| Java Platform, Standard Edition 6 | Java 6 |
| Java Platform, Standard Edition 7 | Java 7 |
| Java Platform, Standard Edition 8 | Java 8 |
| Java Platform, Standard Edition 9 | Java 9 |

Los ejemplos son razonablemente completos, pero favorecen la legibilidad sobre la completitud. Utilizan libremente clases de los paquetes java.util y java.io. Para compilar ejemplos, es posible que tengas que agregar una o más declaraciones de importación u otro código repetitivo similar. El sitio web del libro, http://joshbloch.com/effectivejava, contiene una versión ampliada de cada ejemplo, que puedes compilar y ejecutar.

En su mayor parte, este libro utiliza términos técnicos tal como se definen en The Java Language Specification, Java SE 8 Edition [JLS]. Algunos términos merecen una mención especial. El lenguaje admite cuatro tipos de tipos: interfaces (incluidas las anotaciones), clases (incluidos los enums), arrays y primitivos. Los tres primeros se conocen como tipos de referencia. Las instancias de clase y los arrays son objetos; los valores primitivos no lo son. Los miembros de una clase consisten en sus campos, métodos, clases miembro e interfaces miembro. La firma de un método consiste en su nombre y los tipos de sus parámetros formales; la firma no incluye el tipo de retorno del método.

Este libro usa algunos términos de manera diferente a The Java Language Specification. A diferencia de The Java Language Specification, este libro usa herencia como sinónimo de subclase. En lugar de usar el término herencia para interfaces, este libro simplemente afirma que una clase implementa una interfaz o que una interfaz extiende otra. Para describir el nivel de acceso que se aplica cuando no se especifica ninguno, este libro usa el tradicional package-private en lugar del técnicamente correcto package access [JLS, 6.6.1].

Este libro utiliza algunos términos técnicos que no están definidos en The Java Language Specification. El término API exportada, o simplemente API, se refiere a las clases, interfaces, constructores, miembros y formas serializadas mediante las cuales un programador accede a una clase, interfaz o paquete. (El término API, que es la abreviatura de interfaz de programación de aplicaciones, se usa en preferencia al término interfaz, que sería preferible de otro modo, para evitar confusiones con la construcción del lenguaje de ese nombre.) Un programador que escribe un programa que usa una API se denomina usuario de la API. Una clase cuya implementación usa una API es un cliente de la API.

Las clases, interfaces, constructores, miembros y formas serializadas se conocen colectivamente como elementos de la API. Una API exportada consiste en los elementos de la API que son accesibles fuera del paquete que define la API. Estos son los elementos de la API que cualquier cliente puede usar y que el autor de la API se compromete a soportar. No por coincidencia, estos son también los elementos para los cuales la utilidad Javadoc genera documentación en su modo de operación predeterminado. En términos generales, la API exportada de un paquete consiste en los miembros y constructores públicos y protegidos de cada clase o interfaz pública en el paquete.

En Java 9, se añadió un sistema de módulos a la plataforma. Si una biblioteca hace uso del sistema de módulos, su API exportada es la unión de las APIs exportadas de todos los paquetes exportados por la declaración del módulo de la biblioteca.


[CAPITULO_2](/capitulo_2.md)