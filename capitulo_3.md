[CAPITULO_2](/capitulo_2.md)

# Capítulo 3. Métodos comunes a todos los objetos

Aunque **Object** es una clase concreta, está diseñada principalmente para ser extendida. Todos sus métodos no finales (`equals`, `hashCode`, `toString`, `clone` y `finalize`) tienen contratos generales explícitos porque están diseñados para ser sobrescritos. Es responsabilidad de cualquier clase que sobrescriba estos métodos obedecer sus contratos generales; no hacerlo impedirá que otras clases que dependen de los contratos (como `HashMap` y `HashSet`) funcionen correctamente en conjunto con la clase.

Este capítulo te explica cuándo y cómo sobrescribir los métodos no finales de `Object`. El método `finalize` se omite en este capítulo porque se discutió en el Ítem 8. Aunque no es un método de `Object`, `Comparable.compareTo` se discute en este capítulo porque tiene un carácter similar.

## ÍTEM 10: OBEDECE EL CONTRATO GENERAL AL SOBRESCRIBIR EQUALS

Sobrescribir el método `equals` parece simple, pero hay muchas formas de hacerlo mal, y las consecuencias pueden ser graves. La manera más fácil de evitar problemas es **no sobrescribir el método `equals`**, en cuyo caso cada instancia de la clase es igual solo a sí misma. Esto es lo correcto si se aplica alguna de las siguientes condiciones:

1. **Cada instancia de la clase es inherentemente única**. 
   Esto es cierto para clases como `Thread` que representan entidades activas en lugar de valores. La implementación de `equals` proporcionada por `Object` tiene exactamente el comportamiento correcto para estas clases.

2. **No hay necesidad de que la clase proporcione una prueba de "igualdad lógica"**. 
   Por ejemplo, `java.util.regex.Pattern` podría haber sobrescrito `equals` para verificar si dos instancias de `Pattern` representaban exactamente la misma expresión regular, pero los diseñadores no pensaron que los clientes necesitarían o querrían esta funcionalidad. En estas circunstancias, la implementación de `equals` heredada de `Object` es ideal.

3. **Una superclase ya ha sobrescrito `equals`, y el comportamiento de la superclase es apropiado para esta clase**. 
   Por ejemplo, la mayoría de las implementaciones de `Set` heredan su implementación de `equals` de `AbstractSet`, las implementaciones de `List` de `AbstractList`, y las implementaciones de `Map` de `AbstractMap`.

4. **La clase es privada o package-private, y estás seguro de que su método `equals` nunca será invocado**. 
   Si eres extremadamente adverso al riesgo, puedes sobrescribir el método `equals` para asegurarte de que no se invoque accidentalmente:

   ```java
   @Override public boolean equals(Object o) {
       throw new AssertionError(); // El método nunca se llama
   }
   ```

Puntos clave:

1. Sobrescribir `equals` es propenso a errores y puede tener consecuencias graves.
2. La opción más segura es no sobrescribir `equals` si no es necesario.
3. Hay cuatro situaciones principales en las que no se debe sobrescribir `equals`:
   - Instancias inherentemente únicas
   - No se necesita una prueba de "igualdad lógica"
   - La superclase ya proporciona un `equals` apropiado
   - La clase es privada o package-private y `equals` nunca será invocado
4. En casos de extrema precaución, se puede sobrescribir `equals` para lanzar una excepción si se invoca accidentalmente.

# Cuándo es apropiado sobrescribir equals

Es apropiado sobrescribir `equals` cuando una clase tiene una noción de igualdad lógica que difiere de la mera identidad del objeto y una superclase no ha sobrescrito ya `equals`. Este es generalmente el caso de las **clases de valor**. Una clase de valor es simplemente una clase que representa un valor, como `Integer` o `String`. Un programador que compara referencias a objetos de valor usando el método `equals` espera averiguar si son lógicamente equivalentes, no si se refieren al mismo objeto. Sobrescribir el método `equals` no solo es necesario para satisfacer las expectativas del programador, sino que también permite que las instancias sirvan como claves de mapas o elementos de conjuntos con un comportamiento predecible y deseable.

Un tipo de clase de valor que no requiere que se sobrescriba el método `equals` es una clase que usa control de instancias (Ítem 1) para asegurar que exista como máximo un objeto con cada valor. Los tipos Enum (Ítem 34) caen en esta categoría. Para estas clases, la igualdad lógica es lo mismo que la identidad del objeto, por lo que el método `equals` de Object funciona como un método de igualdad lógica.

## El contrato de equals

Cuando sobrescribes el método `equals`, debes adherirte a su contrato general. Aquí está el contrato, de la especificación de Object:

El método `equals` implementa una relación de equivalencia. Tiene estas propiedades:

1. **Reflexiva**: Para cualquier valor de referencia no nulo x, `x.equals(x)` debe devolver true.

2. **Simétrica**: Para cualquier valor de referencia no nulo x e y, `x.equals(y)` debe devolver true si y solo si `y.equals(x)` devuelve true.

3. **Transitiva**: Para cualquier valor de referencia no nulo x, y, z, si `x.equals(y)` devuelve true y `y.equals(z)` devuelve true, entonces `x.equals(z)` debe devolver true.

4. **Consistente**: Para cualquier valor de referencia no nulo x e y, múltiples invocaciones de `x.equals(y)` deben devolver consistentemente true o consistentemente false, siempre que no se modifique ninguna información utilizada en las comparaciones de equals.

Para cualquier valor de referencia no nulo x, `x.equals(null)` debe devolver false. A menos que tengas inclinaciones matemáticas, esto podría parecer un poco aterrador, ¡pero no lo ignores! Si lo violas, es muy probable que tu programa se comporte erráticamente o se bloquee, y puede ser muy difícil identificar la fuente del fallo. Parafraseando a John Donne, ninguna clase es una isla. Las instancias de una clase se pasan frecuentemente a otra. Muchas clases, incluidas todas las clases de colecciones, dependen de que los objetos que se les pasan obedezcan el contrato de equals.

## Explicación detallada del contrato

Ahora que eres consciente de los peligros de violar el contrato de `equals`, repasemos el contrato en detalle. La buena noticia es que, a pesar de las apariencias, realmente no es muy complicado. Una vez que lo entiendes, no es difícil adherirse a él.

Entonces, ¿qué es una relación de equivalencia? En términos generales, es un operador que divide un conjunto de elementos en subconjuntos cuyos elementos se consideran iguales entre sí. Estos subconjuntos se conocen como clases de equivalencia. Para que un método `equals` sea útil, todos los elementos en cada clase de equivalencia deben ser intercambiables desde la perspectiva del usuario.

Ahora examinemos los cinco requisitos por turno:

1. **Reflexividad**: El primer requisito dice simplemente que un objeto debe ser igual a sí mismo. Es difícil imaginar violar este requisito sin intención. Si lo violaras y luego agregaras una instancia de tu clase a una colección, el método `contains` bien podría decir que la colección no contiene la instancia que acabas de agregar.

2. **Simetría**: El segundo requisito dice que dos objetos cualesquiera deben estar de acuerdo sobre si son iguales. A diferencia del primer requisito, no es difícil imaginar violar este sin intención. Por ejemplo, considera la siguiente clase, que implementa una cadena insensible a mayúsculas y minúsculas. El caso de la cadena se preserva por `toString` pero se ignora en las comparaciones de `equals`:

```java
// ¡Violación del principio de simetría!
public final class CaseInsensitiveString {
    private final String s;

    public CaseInsensitiveString(String s) {
        this.s = Objects.requireNonNull(s);
    }

    // ¡Violación del principio de simetría!
    @Override public boolean equals(Object o) {
        if (o instanceof CaseInsensitiveString)
            return s.equalsIgnoreCase(
                ((CaseInsensitiveString) o).s);
        if (o instanceof String)  // One-way interoperability!
            return s.equalsIgnoreCase((String) o);
        return false;
    }
    ... // Remainder omitted
}
```

El método `equals` bien intencionado en esta clase intenta ingenuamente interoperar con cadenas ordinarias. Supongamos que tenemos una cadena insensible a mayúsculas y minúsculas y una ordinaria:

```java
CaseInsensitiveString cis = new CaseInsensitiveString("Polish");
String s = "polish";
```


Como se esperaba, `cis.equals(s)` devuelve `true`. El problema es que mientras el método `equals` en `CaseInsensitiveString` conoce las cadenas ordinarias, el método `equals` en `String` es ajeno a las cadenas insensibles a mayúsculas y minúsculas. Por lo tanto, `s.equals(cis)` devuelve `false`, una clara violación de la simetría. Supongamos que pones una cadena insensible a mayúsculas y minúsculas en una colección:

```java
List<CaseInsensitiveString> list = new ArrayList<>();
list.add(cis);
```

¿Qué devuelve `list.contains(s)` en este punto? Quién sabe. En la implementación actual de OpenJDK, resulta que devuelve false, pero eso es solo un artefacto de implementación. En otra implementación, podría fácilmente devolver true o lanzar una excepción en tiempo de ejecución. Una vez que has violado el contrato de equals, simplemente no sabes cómo se comportarán otros objetos cuando se enfrenten a tu objeto.

Para eliminar el problema, simplemente elimina el intento mal concebido de interoperar con String del método equals. Una vez que hagas esto, puedes refactorizar el método en una sola declaración de retorno.

```java
@Override public boolean equals(Object o) {
return o instanceof CaseInsensitiveString &&
                            ((CaseInsensitiveString) o).s.equalsIgnoreCase(s);
}
```

2. **Transitividad** El tercer requisito del contrato de equals dice que si un objeto es igual a un segundo y el segundo objeto es igual a un tercero, entonces el primer objeto debe ser igual al tercero. Nuevamente, no es difícil imaginar violar este requisito sin intención. Considera el caso de una subclase que añade un nuevo componente de valor a su superclase. En otras palabras, la subclase añade una pieza de información que afecta las comparaciones de equals. Comencemos con una simple clase inmutable de punto bidimensional entero:

```java
public class Point {
    private final int x;
    private final int y;

    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }

    @Override public boolean equals(Object o) {
        if (!(o instanceof Point))
            return false;
        Point p = (Point)o;
        return p.x == x && p.y == y;
    }
    ... // Remainder omitted
}
```

Supón que quieres extender esta clase, añadiendo la noción de color a un punto:

```java
public class ColorPoint extends Point {
    private final Color color;
    
    public ColorPoint(int x, int y, Color color) {
        super(x, y);
        this.color = color;
    }
    ... // Resto omitido
}
```


¿Cómo debería verse el método equals? Si lo omites por completo, la implementación se hereda de Point y la información de color se ignora en las comparaciones de igualdad. Aunque esto no viola el contrato de equals, claramente es inaceptable. Supongamos que escribes un método equals que devuelve true solo si su argumento es otro punto de color con la misma posición y color:

```js title=""
@Override
public boolean equals(Object o) {
    if (!(o instanceof ColorPoint))
        return false;
    return super.equals(o) && ((ColorPoint) o).color == color;
}
```

Este método primero verifica si el objeto pasado es una instancia de ColorPoint. Luego, llama al método equals de la superclase (Point) para verificar la igualdad de posición y compara los colores directamente utilizando el operador ==.

El problema aquí es que, si bien la comparación de posición se delega correctamente a la superclase, la comparación de color se realiza utilizando el operador ==, que compara referencias de objetos en lugar de sus valores. Esto puede conducir a resultados incorrectos, especialmente si se están utilizando objetos Color con instancias diferentes pero valores iguales. Además, este método no garantiza la simetría de la igualdad, lo que puede provocar resultados inesperados en comparaciones entre objetos ColorPoint.

El problema con este método es que podrías obtener resultados diferentes al comparar un punto con un punto de color y viceversa. La primera comparación ignora el color, mientras que la segunda comparación siempre devuelve falso porque el tipo del argumento es incorrecto. Para ilustrar esto con un ejemplo, creemos un punto y un punto de color:

```js title=""
Point p = new Point(1, 2);
ColorPoint cp = new ColorPoint(1, 2, Color.RED);
```

Entonces, p.equals(cp) devuelve true, mientras que cp.equals(p) devuelve false. Podrías intentar solucionar el problema haciendo que ColorPoint.equals ignore el color al hacer "comparaciones mixtas":

```js title=""
// Broken - violates transitivity!
@Override
public boolean equals(Object o) {
    if (!(o instanceof Point))
        return false;

    // Si o es un Point normal, realiza una comparación sin considerar el color
    if (!(o instanceof ColorPoint))
        return o.equals(this);

    // o es un ColorPoint; realiza una comparación completa
    return super.equals(o) && ((ColorPoint) o).color == color;
}
```

Este enfoque sí proporciona simetría, pero a expensas de la transitividad. Este enfoque sí proporciona simetría al considerar equals entre Point y ColorPoint, ya que ahora p.equals(cp) y cp.equals(p) devolverán el mismo resultado (true o false) según la posición y la igualdad de color. Sin embargo, este enfoque sacrifica la transitividad, como se mencionó anteriormente, ya que la relación de igualdad entre Point y ColorPoint no se mantiene consistente en todas las situaciones posibles. Por lo tanto, aunque se logra la simetría, se viola la transitividad, lo que no cumple con los requisitos del contrato de equals.

```js title=""
ColorPoint p1 = new ColorPoint(1, 2, Color.RED); // Un ColorPoint con posición (1, 2) y color rojo
Point p2 = new Point(1, 2); // Un Point con posición (1, 2)
ColorPoint p3 = new ColorPoint(1, 2, Color.BLUE); // Un ColorPoint con posición (1, 2) y color azul
```

Ahora, p1.equals(p2) y p2.equals(p3) devuelven true, mientras que p1.equals(p3) devuelve false, una clara violación de la transitividad. Las dos primeras comparaciones son "ciegas al color", mientras que la tercera tiene en cuenta el color.

Además, este enfoque puede causar una recursión infinita: Supongamos que hay dos subclases de Point, digamos ColorPoint y SmellPoint, cada una con este tipo de método equals. Entonces, una llamada a myColorPoint.equals(mySmellPoint) lanzará un StackOverflowError.

Entonces, ¿cuál es la solución? Resulta que este es un problema fundamental de relaciones de equivalencia en lenguajes orientados a objetos. No hay forma de extender una clase instanciable y agregar un componente de valor mientras se preserva el contrato de equals, a menos que estés dispuesto a renunciar a los beneficios de la abstracción orientada a objetos. Puede que escuches que puedes extender una clase instanciable y agregar un componente de valor mientras se preserva el contrato de equals usando una prueba getClass en lugar de la prueba instanceof en el método equals.

Este método equals viola el principio de sustitución de Liskov. Aquí está el método proporcionado:

```js title=""
// Broken - violates Liskov substitution principle (page 43)
@Override
public boolean equals(Object o) {
    if (o == null || o.getClass() != getClass())
        return false;
    Point p = (Point) o;
    return p.x == x && p.y == y;
}
```

Este método tiene el efecto de igualar objetos solo si tienen la misma clase de implementación. Esto puede no parecer tan malo, pero las consecuencias son inaceptables:

una instancia de una subclase de Point sigue siendo un Point, y aún debe funcionar como tal, ¡pero falla en hacerlo si tomas este enfoque!

Supongamos que queremos escribir un método para determinar si un punto está en el círculo unitario. Aquí hay una forma en que podríamos hacerlo:

```js title=""
// Inicializa unitCircle para contener todos los Points en el círculo unitario
private static final Set<Point> unitCircle = Set.of(
    new Point(1, 0), new Point(0, 1),
    new Point(-1, 0), new Point(0, -1));

public static boolean onUnitCircle(Point p) {
    return unitCircle.contains(p);
}
```

Si bien esta puede no ser la forma más rápida de implementar la funcionalidad, funciona bien. Supongamos que extendemos Point de alguna manera trivial que no agregue un componente de valor, por ejemplo, haciendo que su constructor lleve un registro de cuántas instancias se han creado:

```js title=""
public class CounterPoint extends Point {
    private static final AtomicInteger counter = new AtomicInteger();

    public CounterPoint(int x, int y) {
        super(x, y);
        counter.incrementAndGet();
    }

    public static int numberCreated() {
        return counter.get();
    }
}
```

Este nuevo tipo de punto, CounterPoint, hereda de Point pero agrega funcionalidad adicional para rastrear el número de instancias creadas. Sin embargo, debido a la implementación defectuosa del método equals en Point, CounterPoint no se comportará correctamente cuando se utilice en contextos donde se espera un Point, como en el método onUnitCircle. Esto viola el principio de sustitución de Liskov.

El principio de sustitución de Liskov establece que cualquier propiedad importante de un tipo también debe cumplirse para todos sus subtipos, de modo que cualquier método escrito para el tipo también funcione igualmente bien en sus subtipos [Liskov87]. Esta es la declaración formal de nuestra afirmación anterior de que una subclase de Point (como CounterPoint) sigue siendo un Point y debe actuar como tal. Pero supongamos que pasamos un CounterPoint al método onUnitCircle. Si la clase Point utiliza un método equals basado en getClass, el método onUnitCircle devolverá false independientemente de las coordenadas x e y de la instancia de CounterPoint. Esto se debe a que la mayoría de las colecciones, incluido el HashSet utilizado por el método onUnitCircle, usan el método equals para probar la contención, y ninguna instancia de CounterPoint es igual a ninguna instancia de Point. Sin embargo, si utilizas un método equals basado en instanceof adecuado en Point, el mismo método onUnitCircle funciona bien cuando se le presenta una instancia de CounterPoint.

Si bien no hay una forma satisfactoria de extender una clase instanciable y agregar un componente de valor, hay una solución elegante: sigue el consejo del Elemento 18, "Prefiere la composición sobre la herencia". En lugar de que ColorPoint extienda Point, dale a ColorPoint un campo privado de tipo Point y un método de vista público (Elemento 6) que devuelva el punto en la misma posición que este punto de color.

```js title=""
import java.awt.Color;
import java.util.Objects;

public class ColorPoint {
    private final Point point;
    private final Color color;

    public ColorPoint(int x, int y, Color color) {
        this.point = new Point(x, y);
        this.color = Objects.requireNonNull(color);
    }

    public Point asPoint() {
        return point;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof ColorPoint)) return false;
        ColorPoint cp = (ColorPoint) o;
        return cp.point.equals(point) && cp.color.equals(color);
    }

    // Remainder omitted
}
```

Este es un excelente ejemplo de cómo utilizar la composición para agregar un componente de valor sin violar el contrato de igualdad. La clase ColorPoint contiene un campo de tipo Point y un campo para el color. Al hacerlo, ColorPoint puede representar un punto en un plano con un color asociado.

El método asPoint() proporciona una vista del punto subyacente sin exponer directamente el campo point, lo que ayuda a encapsular la implementación y a prevenir modificaciones no deseadas.

El método equals compara tanto el punto como el color de dos ColorPoint, asegurando que dos instancias de ColorPoint sean iguales si tienen el mismo punto y el mismo color.

Este enfoque de composición permite agregar funcionalidad adicional (en este caso, el color) a un tipo existente (en este caso, Point) sin heredar directamente de él y sin violar el contrato de igualdad. ¡Es una forma elegante de extender la funcionalidad de las clases existentes!

Hay algunas clases en las bibliotecas de la plataforma Java que extienden una clase instanciable y agregan un componente de valor. Por ejemplo, java.sql.Timestamp extiende java.util.Date y agrega un campo de nanosegundos. La implementación de equals para Timestamp viola la simetría y puede causar un comportamiento errático si objetos Timestamp y Date se utilizan en la misma colección o se mezclan de alguna otra manera. La clase Timestamp tiene una advertencia que alerta a los programadores sobre la mezcla de fechas y marcas de tiempo. Aunque no tendrás problemas mientras los mantengas separados, no hay nada que te impida mezclarlos, y los errores resultantes pueden ser difíciles de depurar. Este comportamiento de la clase Timestamp fue un error y no debe ser emulado.

Es importante destacar que puedes agregar un componente de valor a una subclase de una clase abstracta sin violar el contrato de equals. Esto es relevante para el tipo de jerarquías de clases que se obtienen al seguir el consejo en el Elemento 23, "Prefiere jerarquías de clases a clases etiquetadas". Por ejemplo, podrías tener una clase abstracta Shape sin componentes de valor, una subclase Circle que agrega un campo de radio, y una subclase Rectangle que agrega campos de longitud y ancho. Los problemas del tipo mostrado anteriormente no ocurrirán siempre que sea imposible crear una instancia de la superclase directamente.

`Consistencia` - El cuarto requisito del contrato de equals dice que si dos objetos son iguales, deben permanecer iguales en todo momento a menos que uno (o ambos) se modifique. En otras palabras, los objetos mutables pueden ser iguales a diferentes objetos en diferentes momentos, mientras que los objetos inmutables no. Cuando escribes una clase, piensa detenidamente si debería ser inmutable (Punto 17). Si concluyes que debería serlo, asegúrate de que tu método equals haga cumplir la restricción de que los objetos iguales permanezcan iguales y los objetos desiguales permanezcan desiguales en todo momento.

Ya sea que una clase sea inmutable o no, no escribas un método equals que dependa de recursos no confiables. Es extremadamente difícil satisfacer el requisito de consistencia si violas esta prohibición. Por ejemplo, el método equals de java.net.URL depende de la comparación de las direcciones IP de los hosts asociados con las URL. Traducir un nombre de host a una dirección IP puede requerir acceso a la red, y no está garantizado que produzca los mismos resultados a lo largo del tiempo. Esto puede hacer que el método equals de URL viole el contrato equals y ha causado problemas en la práctica. El comportamiento del método equals de URL fue un gran error y no debe ser emulado. Desafortunadamente, no se puede cambiar debido a los requisitos de compatibilidad. Para evitar este tipo de problemas, los métodos equals solo deben realizar cálculos deterministas en objetos residentes en memoria.

`No nulidad` - El último requisito carece de un nombre oficial, por lo que me he tomado la libertad de llamarlo "no nulidad". Dice que todos los objetos deben ser desiguales a null. Si bien es difícil imaginar que accidentalmente se devuelva true en respuesta a la invocación o.equals(null), no es difícil imaginar que accidentalmente se arroje una NullPointerException. El contrato general lo prohíbe. Muchas clases tienen métodos equals que lo previenen con una prueba explícita para null:

```java title=""
@Override
public boolean equals(Object o) {
    if (o == null)
        return false;
    ... // Resto de la implementación
}
```

Esta prueba es innecesaria. Para verificar la igualdad de su argumento, el método equals debe primero castear su argumento a un tipo apropiado para que se puedan invocar sus accessors o acceder a sus campos. Antes de hacer el cast, el método debe usar el operador instanceof para verificar que su argumento sea del tipo correcto:

```java title=""
@Override
public boolean equals(Object o) {
    if (!(o instanceof MiTipo))
        return false;
    MiTipo mt = (MiTipo) o;
    ... // Implementación del método de igualdad
}
```

Si faltara esta verificación de tipo y el método equals recibiera un argumento del tipo incorrecto, el método equals lanzaría una ClassCastException, lo cual viola el contrato de equals. Pero se especifica que el operador `instanceof` devolverá `false` si su primer operando es `null`, independientemente del tipo que aparezca en el segundo operando [JLS, 15.20.2]. Por lo tanto, la verificación de tipo devolverá `false` si se pasa `null`, por lo que no necesitas una verificación explícita de `null`.

Poniéndolo todo junto, aquí tienes una receta para un método `equals` de alta calidad:

1. `Utiliza el operador == para verificar si el argumento es una referencia a este objeto.` Si es así, devuelve true. Esto es solo una optimización de rendimiento, pero vale la pena hacerlo si la comparación puede ser potencialmente costosa.

2. `Utiliza el operador instanceof para verificar si el argumento tiene el tipo correcto.` Si no es así, devuelve false. Normalmente, el tipo correcto es la clase en la que ocurre el método. Ocasionalmente, puede ser alguna interfaz implementada por esta clase. Utiliza una interfaz si la clase implementa una interfaz que refina el contrato de equals para permitir comparaciones entre clases que implementan la interfaz. Las interfaces de colección como Set, List, Map y Map.Entry tienen esta propiedad.

3. `Haz un casting del argumento al tipo correcto.` Debido a que este casting fue precedido por una prueba de instanceof, se garantiza que tendrá éxito.

4. `Para cada campo "significativo" en la clase, verifica si ese campo del argumento coincide con el campo correspondiente de este objeto.` Si todas estas pruebas tienen éxito, devuelve true; de lo contrario, devuelve false. Si el tipo en el Paso 2 es una interfaz, debes acceder a los campos del argumento mediante métodos de interfaz; si el tipo es una clase, es posible que puedas acceder a los campos directamente, según su accesibilidad. Para campos primitivos cuyo tipo no sea float o double, utiliza el operador == para comparaciones; para campos de referencia a objetos, llama al método equals recursivamente; para campos float, utiliza el método estático Float.compare(float, float); y para campos double, utiliza el método Double.compare(double, double). El tratamiento especial de los campos float y double es necesario debido a la existencia de Float.NaN, -0.0f y los valores double análogos; consulta JLS 15.21.1 o la documentación de Float.equals para obtener más detalles. Aunque podrías comparar campos float y double con los métodos estáticos Float.equals y Double.equals, esto implicaría el autoboxing en cada comparación, lo que tendría un rendimiento deficiente. Para campos de array, aplica estas pautas a cada elemento. Si cada elemento en un campo de array es significativo, utiliza uno de los métodos Arrays.equals. Algunos campos de referencia a objetos pueden contener null de manera legítima. Para evitar la posibilidad de una NullPointerException, verifica dichos campos para la igualdad utilizando el método estático Objects.equals(Object, Object). Para algunas clases, como CaseInsensitiveString mencionada anteriormente, las comparaciones de campos son más complejas que las simples pruebas de igualdad. Si este es el caso, es posible que desees almacenar una forma canónica del campo para que el método equals pueda realizar una comparación exacta económica en formas canónicas en lugar de una comparación no estándar más costosa. Esta técnica es más apropiada para clases inmutables (Ítem 17); si el objeto puede cambiar, debes mantener actualizada la forma canónica. El rendimiento del método equals puede verse afectado por el orden en que se comparan los campos. Para obtener el mejor rendimiento, primero debes comparar campos que es más probable que difieran, menos costosos de comparar, o, idealmente, ambos. No debes comparar campos que no forman parte del estado lógico de un objeto, como los campos de bloqueo utilizados para sincronizar operaciones. No es necesario comparar campos derivados, que pueden calcularse a partir de "campos significativos", pero hacerlo puede mejorar el rendimiento del método equals. Si un campo derivado equivale a una descripción resumida de todo el objeto, comparar este campo te ahorrará el gasto de comparar los datos reales si la comparación falla. Por ejemplo, supongamos que tienes una clase Polygon, y almacenas el área en caché. Si dos polígonos tienen áreas desiguales, no es necesario molestar en comparar sus lados y vértices.

`Cuando hayas terminado de escribir tu método equals, hazte tres preguntas: ¿Es simétrico? ¿Es transitivo? ¿Es consistente?` Y no te limites a preguntarte a ti mismo; escribe pruebas unitarias para verificarlo, a menos que hayas utilizado AutoValue (página 49) para generar tu método equals, en cuyo caso puedes omitir las pruebas de forma segura. Si las propiedades no se cumplen, averigua por qué y modifica el método equals en consecuencia. Por supuesto, tu método equals también debe satisfacer las otras dos propiedades (reflexividad y no nulidad), pero estas dos generalmente se encargan por sí mismas.
Se muestra un método equals construido de acuerdo con la receta anterior en esta clase PhoneNumber simplista:

```java title=""
// Clase con un método equals típico
public final class PhoneNumber {
    private final short areaCode, prefix, lineNum;

    public PhoneNumber(int areaCode, int prefix, int lineNum) {
        this.areaCode = rangeCheck(areaCode, 999, "código de área");
        this.prefix = rangeCheck(prefix, 999, "prefijo");
        this.lineNum = rangeCheck(lineNum, 9999, "número de línea");
    }

    private static short rangeCheck(int val, int max, String arg) {
        if (val < 0 || val > max)
            throw new IllegalArgumentException(arg + ": " + val);
        return (short) val;
    }

    @Override
    public boolean equals(Object o) {
        if (o == this)
            return true;
        if (!(o instanceof PhoneNumber))
            return false;
        PhoneNumber pn = (PhoneNumber)o;
        return pn.lineNum == lineNum && pn.prefix == prefix && pn.areaCode == areaCode;
    }

    // El resto se omite...
}
```

Aquí hay algunas advertencias finales:
- Siempre sobrescribe hashCode cuando sobrescribes equals (Ítem 11).
- No intentes ser demasiado ingenioso. Si simplemente pruebas los campos para la igualdad, no es difícil adherirse al contrato equals. Si eres demasiado agresivo en la búsqueda de equivalencias, es fácil meterse en problemas. Generalmente, es una mala idea tener en cuenta cualquier forma de aliasing. Por ejemplo, la clase File no debería intentar igualar enlaces simbólicos que se refieran al mismo archivo. Afortunadamente, no lo hace.
- No substituyas otro tipo por Object en la declaración de equals. No es raro que un programador escriba un método equals que se vea así y luego pase horas desconcertado sobre por qué no funciona correctamente:

```js title=""
// Broken - parameter type must be Object!
public boolean equals(MyClass o) {
...
}
```

El problema es que este método no sobrescribe Object.equals, cuyo argumento es de tipo Object, sino que lo sobrecarga en su lugar (Ítem 52). Es inaceptable proporcionar un método equals "fuertemente tipado" incluso además del normal, porque puede causar que las anotaciones Override en las subclases generen falsos positivos y proporcionen una falsa sensación de seguridad. El uso consistente de la anotación Override, como se ilustra a lo largo de este ítem, te evitará cometer este error (Ítem 40). Este método equals no se compilará y el mensaje de error te dirá exactamente qué está mal:

```js title=""
// Still broken, but won’t compile
@Override public boolean equals(MyClass o) {
...
}
```

Escribir y probar los métodos equals (y hashCode) es tedioso, y el código resultante es mundano. Una excelente alternativa a escribir y probar estos métodos manualmente es usar el framework de código abierto AutoValue de Google, que genera automáticamente estos métodos para ti, activados por una única anotación en la clase. En la mayoría de los casos, los métodos generados por AutoValue son esencialmente idénticos a los que escribirías tú mismo.
También las IDEs tienen facilidades para generar los métodos equals y hashCode, pero el código fuente resultante es más verboso y menos legible que el código que utiliza AutoValue, no rastrea los cambios en la clase automáticamente y, por lo tanto, requiere pruebas. Dicho esto, tener las IDEs generar los métodos equals (y hashCode) es generalmente preferible a implementarlos manualmente porque las IDEs no cometen errores descuidados, y los humanos sí.
En resumen, no sobrescribas el método equals a menos que sea necesario: en muchos casos, la implementación heredada de Object hace exactamente lo que deseas. Si decides sobrescribir equals, asegúrate de comparar todos los campos significativos de la clase y de compararlos de una manera que preserve las cinco disposiciones del contrato equals.

## Siempre sobrescribe hashCode cuando sobrescribes equals

`Debes sobrescribir hashCode en cada clase que sobrescribe equals.` Si no lo haces, tu clase violará el contrato general para hashCode, lo que evitará que funcione correctamente en colecciones como HashMap y HashSet. Aquí está el contrato, adaptado de la especificación de Object:
- Cuando el método hashCode se invoca en un objeto repetidamente durante la ejecución de una aplicación, debe devolver consistentemente el mismo valor, siempre que no se modifique ninguna información utilizada en comparaciones de equals. Este valor no necesita permanecer consistente de una ejecución de una aplicación a otra.
- Si dos objetos son iguales según el método equals(Object), entonces llamar a hashCode en los dos objetos debe producir el mismo resultado entero.
- Si dos objetos son desiguales según el método equals(Object), no es necesario que llamar a hashCode en cada uno de los objetos produzca resultados distintos. Sin embargo, el programador debe ser consciente de que producir resultados distintos para objetos desiguales puede mejorar el rendimiento de las tablas hash.

La provisión clave que se viola cuando no se sobrescribe hashCode es la segunda: los objetos iguales deben tener códigos hash iguales. Dos instancias distintas pueden ser lógicamente iguales según el método equals de una clase, pero para el método hashCode de Object, son simplemente dos objetos con poco en común. Por lo tanto, el método hashCode de Object devuelve dos números aparentemente aleatorios en lugar de dos números iguales como requiere el contrato.
Por ejemplo, supongamos que intentas usar instancias de la clase PhoneNumber del Ítem 10 como claves en un HashMap:
```
Map<PhoneNumber, String> m = new HashMap<>();
m.put(new PhoneNumber(707, 867, 5309), "Jenny");
```

En este punto, podrías esperar que m.get(new PhoneNumber(707, 867, 5309)) devuelva "Jenny", pero en cambio, devuelve null. Observa que están involucradas dos instancias de PhoneNumber: una se utiliza para insertar en el HashMap y una segunda, igual, se utiliza para (intentar) recuperar. El fallo de la clase PhoneNumber para sobrescribir hashCode hace que las dos instancias iguales tengan códigos hash desiguales, en violación del contrato de hashCode. Por lo tanto, es probable que el método get busque el número de teléfono en un bucket hash diferente al que fue almacenado por el método put. Incluso si las dos instancias suceden a tener el mismo hash bucket, es casi seguro que el método get devolverá null, porque HashMap tiene una optimización que almacena en caché el código hash asociado con cada entrada y no se molesta en comprobar la igualdad de objetos si los códigos hash no coinciden.


Arreglar este problema es tan simple como escribir un método `hashCode` apropiado para `PhoneNumber`. Entonces, ¿cómo debería ser un método `hashCode`? Es trivial escribir uno malo. Este, por ejemplo, es siempre legal pero nunca debería usarse:

```java
// La peor implementación legal posible de hashCode - ¡nunca la uses!
@Override public int hashCode() { return 42; }
```


Es legal porque asegura que objetos iguales tengan el mismo código hash. Es atroz porque asegura que cada objeto tiene el mismo código hash. Por lo tanto, cada objeto se asigna al mismo bucket, y las tablas hash degeneran en listas enlazadas. Los programas que deberían ejecutarse en tiempo lineal en su lugar se ejecutan en tiempo cuadrático. Para tablas hash grandes, esta es la diferencia entre funcionar y no funcionar.
Una buena función hash tiende a producir códigos hash desiguales para instancias desiguales. Esto es exactamente lo que significa la tercera parte del contrato de hashCode. Idealmente, una función hash debería distribuir cualquier colección razonable de instancias desiguales uniformemente a través de todos los valores int. Lograr este ideal puede ser difícil. Afortunadamente, no es muy difícil lograr una aproximación justa. Aquí hay una receta simple:

1. Declara una variable int llamada result, e inicialízala con el código hash c para el primer campo significativo en tu objeto, como se calcula en el paso 2.a. (Recuerda del Ítem 10 que un campo significativo es un campo que afecta las comparaciones de equals).

2. Para cada campo significativo restante f en tu objeto, haz lo siguiente:

a. Calcula un código hash int c para el campo:

i. Si el campo es de un tipo primitivo, calcula Type.hashCode(f), donde Type es la clase primitiva encapsulada correspondiente al tipo de f.

ii. Si el campo es una referencia a un objeto y el método equals de esta clase compara el campo invocando recursivamente equals, invoca recursivamente hashCode en el campo. Si se requiere una comparación más compleja, calcula una "representación canónica" para este campo e invoca hashCode en la representación canónica. Si el valor del campo es null, usa 0 (o alguna otra constante, pero 0 es tradicional).

iii. Si el campo es un array, trátalo como si cada elemento significativo fuera un campo separado. Es decir, calcula un código hash para cada elemento significativo aplicando estas reglas recursivamente, y combina los valores según el paso 2.b. Si el array no tiene elementos significativos, usa una constante, preferiblemente no 0. Si todos los elementos son significativos, usa Arrays.hashCode.

b. Combina el código hash c calculado en el paso 2.a en result de la siguiente manera:
result = 31 * result + c;
Devuelve result.

3. Devuelve result.
Cuando hayas terminado de escribir el método hashCode, pregúntate si las instancias iguales tienen códigos hash iguales. Escribe pruebas unitarias para verificar tu intuición (a menos que hayas usado AutoValue para generar tus métodos equals y hashCode, en cuyo caso puedes omitir estas pruebas con seguridad). Si las instancias iguales tienen códigos hash desiguales, averigua por qué y soluciona el problema.

Puedes excluir campos derivados del cálculo del código hash. En otras palabras, puedes ignorar cualquier campo cuyo valor pueda calcularse a partir de campos incluidos en el cálculo. Debes excluir cualquier campo que no se use en las comparaciones de equals, o corres el riesgo de violar la segunda disposición del contrato de hashCode.

La multiplicación en el paso 2.b hace que el resultado dependa del orden de los campos, produciendo una función hash mucho mejor si la clase tiene múltiples campos similares. Por ejemplo, si se omitiera la multiplicación de una función hash de String, todos los anagramas tendrían códigos hash idénticos. Se eligió el valor 31 porque es un número primo impar. Si fuera par y la multiplicación se desbordara, se perdería información, porque la multiplicación por 2 es equivalente a un desplazamiento. La ventaja de usar un número primo es menos clara, pero es tradicional. Una buena propiedad del 31 es que la multiplicación puede reemplazarse por un desplazamiento y una resta para un mejor rendimiento en algunas arquitecturas: 31 * i == (i << 5) - i. Las VMs modernas hacen este tipo de optimización automáticamente.

Apliquemos la receta anterior a la clase PhoneNumber:

```java
// Método hashCode típico
@Override public int hashCode() {
    int result = Short.hashCode(areaCode);
    result = 31 * result + Short.hashCode(prefix);
    result = 31 * result + Short.hashCode(lineNum);
    return result;
}
```

Dado que este método devuelve el resultado de un cálculo determinista simple cuyas únicas entradas son los tres campos significativos en una instancia de PhoneNumber, está claro que las instancias iguales de PhoneNumber tienen códigos hash iguales. Este método es, de hecho, una implementación de hashCode perfectamente buena para PhoneNumber, a la par con las de las bibliotecas de la plataforma Java. Es simple, razonablemente rápido y hace un trabajo razonable dispersando números de teléfono desiguales en diferentes buckets de hash.
Aunque la receta en este ítem produce funciones hash razonablemente buenas, no son de última generación. Son comparables en calidad a las funciones hash encontradas en los tipos de valor de las bibliotecas de la plataforma Java y son adecuadas para la mayoría de los usos. Si tienes una necesidad genuina de funciones hash con menos probabilidad de producir colisiones, consulta com.google.common.hash.Hashing de Guava [Guava].
La clase Objects tiene un método estático que toma un número arbitrario de objetos y devuelve un código hash para ellos. Este método, llamado hash, te permite escribir métodos hashCode de una línea cuya calidad es comparable a los escritos según la receta en este ítem. Desafortunadamente, se ejecutan más lentamente porque implican la creación de arrays para pasar un número variable de argumentos, así como boxing y unboxing si alguno de los argumentos es de tipo primitivo. Este estilo de función hash se recomienda solo en situaciones donde el rendimiento no es crítico. Aquí hay una función hash para PhoneNumber escrita usando esta técnica:

```java
// Método hashCode de una línea - rendimiento mediocre
@Override public int hashCode() {
    return Objects.hash(lineNum, prefix, areaCode);
}
```

Si una clase es inmutable y el costo de calcular el código hash es significativo, podrías considerar almacenar en caché el código hash en el objeto en lugar de recalcularlo cada vez que se solicita. Si crees que la mayoría de los objetos de este tipo se usarán como claves de hash, entonces deberías calcular el código hash cuando se crea la instancia. De lo contrario, podrías optar por inicializar perezosamente el código hash la primera vez que se invoca hashCode. Se requiere cierto cuidado para asegurar que la clase permanezca segura para subprocesos en presencia de un campo inicializado perezosamente (Ítem 83). Nuestra clase PhoneNumber no merece este tratamiento, pero solo para mostrarte cómo se hace, aquí está. Ten en cuenta que el valor inicial para el campo hashCode (en este caso, 0) no debe ser el código hash de una instancia comúnmente creada:

```java
// Método hashCode con código hash en caché inicializado perezosamente
private int hashCode; // Automáticamente inicializado a 0

@Override public int hashCode() {
    int result = hashCode;
    if (result == 0) {
        result = Short.hashCode(areaCode);
        result = 31 * result + Short.hashCode(prefix);
        result = 31 * result + Short.hashCode(lineNum);
        hashCode = result;
    }
    return result;
}
```

No te sientas tentado a excluir campos significativos del cálculo del código hash para mejorar el rendimiento. Aunque la función hash resultante puede ejecutarse más rápido, su pobre calidad puede degradar el rendimiento de las tablas hash hasta el punto de volverlas inutilizables. En particular, la función hash puede enfrentarse a una gran colección de instancias que difieren principalmente en regiones que has elegido ignorar. Si esto sucede, la función hash mapeará todas estas instancias a unos pocos códigos hash, y los programas que deberían ejecutarse en tiempo lineal se ejecutarán en tiempo cuadrático.
Esto no es solo un problema teórico. Antes de Java 2, la función hash de String usaba como máximo dieciséis caracteres espaciados uniformemente a lo largo de la cadena, comenzando con el primer carácter. Para grandes colecciones de nombres jerárquicos, como URLs, esta función mostraba exactamente el comportamiento patológico descrito anteriormente.
No proporciones una especificación detallada para el valor devuelto por hashCode, para que los clientes no puedan depender razonablemente de él; esto te da la flexibilidad para cambiarlo. Muchas clases en las bibliotecas de Java, como String e Integer, especifican el valor exacto devuelto por su método hashCode como una función del valor de la instancia. Esto no es una buena idea sino un error con el que nos vemos obligados a vivir: impide la capacidad de mejorar la función hash en futuras versiones. Si dejas los detalles sin especificar y se encuentra un defecto en la función hash o se descubre una mejor función hash, puedes cambiarla en una versión posterior.
En resumen, debes sobrescribir hashCode cada vez que sobrescribas equals, o tu programa no se ejecutará correctamente. Tu método hashCode debe obedecer el contrato general especificado en Object y debe hacer un trabajo razonable asignando códigos hash desiguales a instancias desiguales. Esto es fácil de lograr, aunque un poco tedioso, usando la receta en la página 51. Como se mencionó en el Ítem 10, el framework AutoValue proporciona una buena alternativa a escribir métodos equals y hashCode manualmente, y los IDEs también proporcionan parte de esta funcionalidad.

## Siempre sobrescribe toString

Aunque Object proporciona una implementación del método toString, la cadena que devuelve generalmente no es lo que el usuario de tu clase quiere ver. Consiste en el nombre de la clase seguido de un signo "arroba" (@) y la representación hexadecimal sin signo del código hash, por ejemplo, PhoneNumber@163b91. El contrato general para toString dice que la cadena devuelta debe ser "una representación concisa pero informativa que sea fácil de leer para una persona". Aunque se podría argumentar que PhoneNumber@163b91 es conciso y fácil de leer, no es muy informativo en comparación con 707-867-5309. El contrato de toString continúa diciendo: "Se recomienda que todas las subclases sobrescriban este método". ¡Un buen consejo, sin duda!

Aunque no es tan crítico como obedecer los contratos de equals y hashCode (Ítems 10 y 11), proporcionar una buena implementación de toString hace que tu clase sea mucho más agradable de usar y facilita la depuración de los sistemas que utilizan la clase.

El método toString se invoca automáticamente cuando un objeto se pasa a println, printf, el operador de concatenación de cadenas, o assert, o es impreso por un depurador. Incluso si nunca llamas a toString en un objeto, otros pueden hacerlo. Por ejemplo, un componente que tiene una referencia a tu objeto puede incluir la representación en cadena del objeto en un mensaje de error registrado. Si no sobrescribes toString, el mensaje puede ser prácticamente inútil.

Si has proporcionado un buen método toString para PhoneNumber, generar un mensaje de diagnóstico útil es tan fácil como esto:

```java
System.out.println("Failed to connect to " + phoneNumber);
```

Los programadores generarán mensajes de diagnóstico de esta manera ya sea que sobrescribas toString o no, pero los mensajes no serán útiles a menos que lo hagas. Los beneficios de proporcionar un buen método toString se extienden más allá de las instancias de la clase a objetos que contienen referencias a estas instancias, especialmente colecciones. ¿Qué preferirías ver al imprimir un mapa, {Jenny=PhoneNumber@163b91} o {Jenny=707-867-5309}?
Cuando sea práctico, el método toString debe devolver toda la información interesante contenida en el objeto, como se muestra en el ejemplo del número de teléfono. Es poco práctico si el objeto es grande o si contiene un estado que no es propicio para la representación en cadena. En estas circunstancias, toString debe devolver un resumen como "Directorio telefónico residencial de Manhattan (1487536 listados)" o "Thread[main,5,main]". Idealmente, la cadena debería ser autoexplicativa. (El ejemplo de Thread no pasa esta prueba.) Una penalización particularmente molesta por no incluir toda la información interesante de un objeto en su representación en cadena son los informes de fallos de prueba que se ven así:
```java
Assertion failure: expected {abc, 123}, but was {abc, 123}.
```

Una decisión importante que tendrás que tomar al implementar un método toString es si especificar el formato del valor de retorno en la documentación. Se recomienda que lo hagas para clases de valor, como número de teléfono o matriz. La ventaja de especificar el formato es que sirve como una representación estándar, inequívoca y legible por humanos del objeto. Esta representación se puede usar para entrada y salida y en objetos de datos persistentes legibles por humanos, como archivos CSV. Si especificas el formato, generalmente es una buena idea proporcionar una fábrica estática o constructor correspondiente para que los programadores puedan traducir fácilmente de ida y vuelta entre el objeto y su representación en cadena. Este enfoque es adoptado por muchas clases de valor en las bibliotecas de la plataforma Java, incluyendo BigInteger, BigDecimal y la mayoría de las clases primitivas encapsuladas.
La desventaja de especificar el formato del valor de retorno de toString es que una vez que lo has especificado, estás atado a él de por vida, suponiendo que tu clase sea ampliamente utilizada. Los programadores escribirán código para analizar la representación, generarla e integrarla en datos persistentes. Si cambias la representación en una versión futura, romperás su código y datos, y se quejarán. Al elegir no especificar un formato, conservas la flexibilidad de agregar información o mejorar el formato en una versión posterior.
Ya sea que decidas especificar el formato o no, debes documentar claramente tus intenciones. Si especificas el formato, debes hacerlo con precisión. Por ejemplo, aquí hay un método toString para ir con la clase PhoneNumber en el Ítem 11:

```java
/**
 * Devuelve la representación en cadena de este número de teléfono.
 * La cadena consta de doce caracteres cuyo formato es
 * "XXX-YYY-ZZZZ", donde XXX es el código de área, YYY es el
 * prefijo, y ZZZZ es el número de línea. Cada una de las letras
 * mayúsculas representa un solo dígito decimal.
 *
 * Si alguna de las tres partes de este número de teléfono es demasiado
 * pequeña para llenar su campo, el campo se rellena con ceros a la izquierda.
 * Por ejemplo, si el valor del número de línea es 123, los últimos
 * cuatro caracteres de la representación en cadena serán "0123".
 */
@Override public String toString() {
    return String.format("%03d-%03d-%04d",
        areaCode, prefix, lineNum);
}
```

```java
Si decides no especificar un formato, el comentario de documentación debería decir algo como esto:

/**
 * Devuelve una breve descripción de esta poción. Los detalles exactos
 * de la representación no están especificados y están sujetos a cambios,
 * pero lo siguiente puede considerarse típico:
 *
 * "[Poción #9: tipo=amor, olor=trementina, aspecto=tinta india]"
 */
@Override public String toString() { ... }
```

Después de leer este comentario, los programadores que produzcan código o datos persistentes que dependan de los detalles del formato no tendrán a nadie más que a sí mismos para culpar cuando el formato cambie.
Ya sea que especifiques el formato o no, proporciona acceso programático a la información contenida en el valor devuelto por toString. Por ejemplo, la clase PhoneNumber debería contener accesores para el código de área, el prefijo y el número de línea. Si no haces esto, obligas a los programadores que necesitan esta información a analizar la cadena. Además de reducir el rendimiento y hacer un trabajo innecesario para los programadores, este proceso es propenso a errores y resulta en sistemas frágiles que se rompen si cambias el formato. Al no proporcionar accesores, conviertes el formato de la cadena en una API de facto, incluso si has especificado que está sujeta a cambios.
No tiene sentido escribir un método toString en una clase de utilidad estática (Ítem 4). Tampoco deberías escribir un método toString en la mayoría de los tipos enum (Ítem 34) porque Java te proporciona uno perfectamente bueno. Sin embargo, deberías escribir un método toString en cualquier clase abstracta cuyos subclases compartan una representación común en cadena. Por ejemplo, los métodos toString en la mayoría de las implementaciones de colecciones se heredan de las clases de colección abstractas.
La instalación de código abierto AutoValue de Google, discutida en el Ítem 10, generará un método toString para ti, al igual que la mayoría de los IDEs. Estos métodos son excelentes para decirte el contenido de cada campo, pero no están especializados en el significado de la clase. Por lo tanto, por ejemplo, sería inapropiado usar un método toString generado automáticamente para nuestra clase PhoneNumber (ya que los números de teléfono tienen una representación estándar en cadena), pero sería perfectamente aceptable para nuestra clase Potion. Dicho esto, un método toString generado automáticamente es muy preferible al heredado de Object, que no te dice nada sobre el valor de un objeto.
Para recapitular, sobrescribe la implementación toString de Object en cada clase instanciable que escribas, a menos que una superclase ya lo haya hecho. Hace que las clases sean mucho más agradables de usar y ayuda en la depuración. El método toString debe devolver una descripción concisa y útil del objeto, en un formato estéticamente agradable.

[CAPITULO_4](/capitulo_4.md)