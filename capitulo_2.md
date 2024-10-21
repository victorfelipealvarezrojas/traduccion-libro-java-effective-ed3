[CAPITULO_1](/README.md)

# Capítulo 2. CREACIÓN Y DESTRUCCIÓN DE OBJETOS
Este capítulo trata sobre la creación y destrucción de objetos: cuándo y cómo crearlos,
cuándo y cómo evitar crearlos, cómo asegurarse de que se destruyan de manera oportuna
y cómo gestionar cualquier acción de limpieza que deba preceder a su destrucción.

## Considera métodos de fábrica estáticos en lugar de constructores

La forma tradicional de que una clase permita a un cliente obtener una instancia es
proporcionar un constructor público. Hay otra técnica que debería ser parte del
kit de herramientas de todo programador. Una clase puede proporcionar un método
de fábrica estático público, que es simplemente un método estático que devuelve una
instancia de la clase. Aquí hay un ejemplo simple de Boolean (la clase primitiva
envuelta para boolean). Este método traduce un valor primitivo boolean en una
referencia de objeto Boolean:

```java
public static Boolean valueOf(boolean b) {
    return b ? Boolean.TRUE : Boolean.FALSE;
}
```
Nótese que un método de fábrica estático no es lo mismo que el patrón Factory Method
de Design Patterns [Gamma95]. El método de fábrica estático descrito en este
elemento no tiene un equivalente directo en Design Patterns.

Una clase puede proporcionar a sus clientes métodos de fábrica estáticos en lugar de,
o además de, constructores públicos. Proporcionar un método de fábrica estático en lugar
de un constructor público tiene tanto ventajas como desventajas.

>Una ventaja de los métodos de fábrica estáticos es que, a diferencia de los constructores,
tienen nombres. Si los parámetros de un constructor no describen por sí mismos el objeto
que se devuelve, un método de fábrica estático con un nombre bien elegido es más fácil
de usar y el código del cliente resultante es más fácil de leer. Por ejemplo, el
constructor `BigInteger(int, int, Random)`, que devuelve un `BigInteger` que
probablemente sea primo, habría sido mejor expresado como un método de fábrica estático
llamado `BigInteger.probablePrime`. (Este método se agregó en Java 4).

Una clase solo puede tener un único constructor con una firma dada. Se sabe que los
programadores han eludido esta restricción proporcionando dos constructores cuyas listas
de parámetros solo difieren en el orden de sus tipos de parámetros. Esta es una idea
realmente mala. El usuario de tal API nunca podrá recordar qué constructor es cuál y
terminará llamando al incorrecto por error. Las personas que lean código que usa estos
constructores no sabrán qué hace el código sin referirse a la documentación de la clase.
Debido a que tienen nombres, los métodos de fábrica estáticos no comparten la restricción
discutida en el párrafo anterior. En casos donde una clase parece requerir múltiples
constructores con la misma firma, reemplace los constructores con métodos de fábrica
estáticos y nombres cuidadosamente elegidos para resaltar sus diferencias.

>Una segunda ventaja de los métodos de fábrica estáticos es que, a diferencia de los
constructores, no están obligados a crear un objeto nuevo cada vez que se invocan. Esto
permite a las clases inmutables (Elemento 17) usar instancias preconstruidas, o almacenar
en caché instancias a medida que se construyen, y dispensarlas repetidamente para evitar
crear objetos duplicados innecesarios. El método `Boolean.valueOf(boolean)` ilustra
esta técnica: nunca crea un objeto. Esta técnica es similar al patrón Flyweight [Gamma95].
Puede mejorar enormemente el rendimiento si se solicitan objetos equivalentes con frecuencia,
especialmente si son costosos de crear.

La capacidad de los métodos de fábrica estáticos para devolver el mismo objeto de
invocaciones repetidas permite a las clases mantener un control estricto sobre qué instancias
existen en cualquier momento. A las clases que hacen esto se les dice que están controladas
por instancias. Hay varias razones para escribir clases controladas por instancias. El control
de instancias permite a una clase garantizar que es un singleton (Elemento 3) o no
instantiable (Elemento 4). Además, permite a una clase de valor inmutable (Elemento 17)
garantizar que no existen dos instancias iguales: a.equals(b) si y solo si a == b. Esto
es la base del patrón Flyweight [Gamma95]. Los tipos Enum (Elemento 34) proporcionan
esta garantía.

>Una tercera ventaja de los métodos de fábrica estáticos es que, a diferencia de los
constructores, pueden devolver un objeto de cualquier subtipo de su tipo de retorno. Esto
te da gran flexibilidad en la elección de la clase del objeto devuelto.

Una aplicación de esta flexibilidad es que una API puede devolver objetos sin hacer públicas
sus clases. Ocultar clases de implementación de esta manera conduce a una API muy compacta.
Esta técnica se presta a marcos basados en interfaces (Elemento 20), donde las interfaces
proporcionan tipos de retorno naturales para los métodos de fábrica estáticos.

Antes de Java 8, las interfaces no podían tener métodos estáticos. Por convención, los
métodos de fábrica estáticos para una interfaz llamada Tipo se ponían en una clase
compañera no instantiable (Elemento 4) llamada Tipos. Por ejemplo, el Java Collections
Framework tiene cuarenta y cinco implementaciones de utilidad de sus interfaces,
proporcionando colecciones inmodificables, colecciones sincronizadas y similares.
Casi todas estas implementaciones se exportan a través de métodos de fábrica estáticos
en una clase no instantiable (java.util.Collections). Las clases de los objetos devueltos
son todas no públicas.

El API del Framework de Colecciones es mucho más pequeña de lo que habría sido si
hubiera exportado cuarenta y cinco clases públicas separadas, una para cada implementación
de conveniencia. No es solo el volumen del API lo que se reduce, sino también el peso
conceptual: el número y la dificultad de los conceptos que los programadores deben dominar
para usar el API. El programador sabe que el objeto devuelto tiene precisamente el API
especificado por su interfaz, por lo que no es necesario leer documentación adicional de
la clase de implementación. Además, el uso de tal método de fábrica estático requiere que
el cliente se refiera al objeto devuelto por la interfaz en lugar de la clase de implementación,
lo cual es generalmente una buena práctica (Elemento 64).

A partir de Java 8, la restricción de que las interfaces no puedan contener métodos estáticos fue
eliminada, por lo que normalmente hay pocas razones para proporcionar una clase compañera no
instantiable para una interfaz. Muchos miembros estáticos públicos que habrían estado en
dicha clase deben en cambio colocarse en la interfaz misma. Sin embargo, cabe señalar que
todavía puede ser necesario poner la mayor parte del código de implementación detrás de estos
métodos estáticos en una clase separada de paquete privado. Esto se debe a que Java 8
requiere que todos los miembros estáticos de una interfaz sean públicos. Java 9 permite
métodos estáticos privados, pero los campos estáticos y las clases de miembros estáticos
todavía deben ser públicos.

>Una cuarta ventaja de las fábricas estáticas es que la clase del objeto devuelto puede variar
de llamada en llamada como una función de los parámetros de entrada. Cualquier subtipo del
tipo de retorno declarado es permisible. La clase del objeto devuelto también puede variar de
versión en versión.

>La clase `EnumSet` (Elemento 36) no tiene constructores públicos, solo fábricas estáticas.
En la implementación de OpenJDK, devuelven una instancia de una de dos subclases,
dependiendo del tamaño del tipo enum subyacente: si tiene sesenta y cuatro o menos
elementos, como la mayoría de los tipos enum, las fábricas estáticas devuelven una instancia
de `RegularEnumSet`, respaldada por un solo long; si el tipo enum tiene sesenta y cinco o
más elementos, las fábricas devuelven una instancia de `JumboEnumSet`, respaldada por un
array de long. La existencia de estas dos clases de implementación es invisible para los
clientes. Si `RegularEnumSet` dejara de ofrecer ventajas de rendimiento para tipos enum
pequeños, podría eliminarse en una futura versión sin efectos negativos. De manera similar,
una futura versión podría agregar una tercera o cuarta implementación de `EnumSet` si
resultara beneficioso para el rendimiento. Los clientes ni saben ni les importa la clase del
objeto que obtienen de la fábrica; solo les importa que sea alguna subclase de `EnumSet`.

>Una quinta ventaja de las fábricas estáticas es que la clase del objeto devuelto no necesita
existir cuando se escribe la clase que contiene el método. Tales métodos de fábrica estáticos
flexibles forman la base de marcos de proveedores de servicios, como la API de Conectividad
de Bases de Datos de Java (JDBC). Un marco de proveedor de servicios es un sistema en el
que los proveedores implementan un servicio y el sistema hace disponibles las
implementaciones a los clientes, desacoplando a los clientes de las implementaciones.

Hay tres componentes esenciales en un marco de proveedor de servicios: una interfaz de
servicio, que representa una implementación; una API de registro de proveedores, que los
proveedores utilizan para registrar implementaciones; y una API de acceso al servicio, que los
clientes usan para obtener instancias del servicio. La API de acceso al servicio puede permitir
a los clientes especificar criterios para elegir una implementación. En ausencia de tales
criterios, la API devuelve una instancia de una implementación predeterminada, o permite al
cliente recorrer todas las implementaciones disponibles. La API de acceso al servicio es la
fábrica estática flexible que forma la base del marco de proveedor de servicios.

Un cuarto componente opcional de un marco de proveedor de servicios es una interfaz de
proveedor de servicios, que describe un objeto de fábrica que produce instancias de la interfaz
de servicio. En ausencia de una interfaz de proveedor de servicios, las implementaciones deben
instanciarse reflexivamente (Elemento 65). En el caso de JDBC, `Connection` desempeña el
papel de la interfaz de servicio, `DriverManager.registerDriver` es la API de registro de
proveedores, `DriverManager.getConnection` es la API de acceso al servicio, y `Driver` es
la interfaz de proveedor de servicios.

Hay muchas variantes del patrón de marco de proveedor de servicios. Por ejemplo, la API de
acceso al servicio puede devolver una interfaz de servicio más rica a los clientes que la
proporcionada por los proveedores. Este es el patrón Bridge [Gamma95]. Los marcos de
inyección de dependencias (Elemento 5) pueden verse como proveedores de servicios
poderosos. Desde Java 6, la plataforma incluye un marco de proveedor de servicios de propósito
general, `java.util.ServiceLoader`, por lo que generalmente no deberías escribir tu propio
(Elemento 59). JDBC no utiliza `ServiceLoader`, ya que el primero es anterior al segundo.

La principal limitación de proporcionar solo métodos de fábrica estáticos es que las clases sin
constructores públicos o protegidos no pueden ser subclasificadas. Por ejemplo, es imposible
subclasificar cualquiera de las clases de implementación de conveniencia en el Framework de
Colecciones. Se podría argumentar que esto es una bendición disfrazada porque alienta a los
programadores a usar la composición en lugar de la herencia (Elemento 18) y es necesario
para tipos inmutables (Elemento 17).

## Considera un builder cuando te enfrentes a muchos parámetros de constructor
Las fábricas estáticas y los constructores comparten una limitación: no escalan bien a un
gran número de parámetros opcionales. Considera el caso de una clase que representa la
etiqueta de Información Nutricional que aparece en los alimentos envasados. Estas etiquetas
tienen algunos campos obligatorios: tamaño de la porción, porciones por envase y calorías por
porción, y más de veinte campos opcionales: grasa total, grasa saturada, grasa trans, colesterol,
sodio, etc. La mayoría de los productos solo tienen valores distintos de cero para unos pocos
de estos campos opcionales.

¿Qué tipo de constructores o fábricas estáticas deberías escribir para tal clase?
Tradicionalmente, los programadores han utilizado el patrón de constructor telescópico, en
el que proporcionas un constructor solo con los parámetros obligatorios, otro con un único
parámetro opcional, un tercero con dos parámetros opcionales, y así sucesivamente, culminando
en un constructor con todos los parámetros opcionales. Así es cómo se ve en la práctica. Por
cuestiones de brevedad, solo se muestran cuatro campos opcionales:

```java
// Patrón de constructor telescópico - ¡no escala bien!
public class NutritionFacts {
    private final int servingSize; // (mL) requerido
    private final int servings; // (por envase) requerido
    private final int calories; // (por porción) opcional
    private final int fat; // (g/porción) opcional
    private final int sodium; // (mg/porción) opcional
    private final int carbohydrate; // (g/porción) opcional

    public NutritionFacts(int servingSize, int servings) {
        this(servingSize, servings, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories) {
        this(servingSize, servings, calories, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories, int fat) {
        this(servingSize, servings, calories, fat, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories, int fat, int sodium) {
        this(servingSize, servings, calories, fat, sodium, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories, int fat, int sodium, int carbohydrate) {
        this.servingSize = servingSize;
        this.servings = servings;
        this.calories = calories;
        this.fat = fat;
        this.sodium = sodium;
        this.carbohydrate = carbohydrate;
    }
}
```
Cuando quieres crear una instancia, usas el constructor con la lista de parámetros más corta
que contenga todos los parámetros que deseas establecer:

```java
NutritionFacts cocaCola = new NutritionFacts(240, 8, 100, 0, 35, 27);
```
Típicamente, esta invocación del constructor requerirá muchos parámetros que no deseas 
establecer, pero te ves obligado a pasar un valor para ellos de todos modos. En este caso, 
pasamos un valor de 0 para la grasa. Con "solo" seis parámetros esto puede no parecer tan malo, 
pero rápidamente se complica a medida que aumenta el número de parámetros.

En resumen, el patrón de constructor telescópico funciona, pero es difícil escribir código de 
cliente cuando hay muchos parámetros, y aún más difícil leerlo. El lector se queda preguntándose 
qué significan todos esos valores y debe contar cuidadosamente los parámetros para averiguarlo. 
Largas secuencias de parámetros del mismo tipo pueden causar errores sutiles. Si el cliente 
invierte accidentalmente dos de estos parámetros, el compilador no se quejará, pero el programa 
se comportará mal en tiempo de ejecución (Elemento 51).

Una segunda alternativa cuando te enfrentas a muchos parámetros opcionales en un 
constructor es el patrón JavaBeans, en el que llamas a un constructor sin parámetros para crear 
el objeto y luego llamas a métodos setter para establecer cada parámetro obligatorio y cada 
parámetro opcional de interés:
```java
// Patrón JavaBeans - permite inconsistencia, exige mutabilidad
public class NutritionFacts {
    // Parámetros inicializados a valores predeterminados (si los hay)
    private int servingSize = -1; // Obligatorio; sin valor predeterminado
    private int servings = -1; // Obligatorio; sin valor predeterminado
    private int calories = 0;
    private int fat = 0;
    private int sodium = 0;
    private int carbohydrate = 0;

    public NutritionFacts() { }

    // Setters
    public void setServingSize(int val) { servingSize = val; }
    public void setServings(int val) { servings = val; }
    public void setCalories(int val) { calories = val; }
    public void setFat(int val) { fat = val; }
    public void setSodium(int val) { sodium = val; }
    public void setCarbohydrate(int val) { carbohydrate = val; }
}
```
Este patrón no tiene ninguna de las desventajas del patrón de constructor telescópico.
Es fácil, aunque un poco verboso, crear instancias y fácil leer el código resultante:

```java
NutritionFacts cocaCola = new NutritionFacts();
cocaCola.setServingSize(240);
cocaCola.setServings(8);
cocaCola.setCalories(100);
cocaCola.setSodium(35);
cocaCola.setCarbohydrate(27);
```

Lamentablemente, el patrón JavaBeans tiene serias desventajas propias.
Debido a que la construcción se divide en múltiples llamadas, un JavaBean puede estar en un
estado inconsistente durante su construcción. La clase no tiene la opción de imponer
consistencia simplemente verificando la validez de los parámetros del constructor. Intentar
usar un objeto cuando está en un estado inconsistente puede causar fallos que están muy
alejados del código que contiene el error y, por lo tanto, son difíciles de depurar. Una
desventaja relacionada es que el patrón JavaBeans impide la posibilidad de hacer una clase
inmutable (Elemento 17) y requiere un esfuerzo adicional por parte del programador para
garantizar la seguridad en el manejo de hilos.

Es posible reducir estas desventajas mediante el "congelamiento" manual del objeto cuando su
construcción está completa y no permitiendo su uso hasta que esté congelado, pero esta
variante es engorrosa y raramente se utiliza en la práctica. Además, puede causar errores en
tiempo de ejecución porque el compilador no puede asegurar que el programador llame al
método de congelamiento en un objeto antes de usarlo.

Afortunadamente, hay una tercera alternativa que combina la seguridad del patrón de
constructor telescópico con la legibilidad del patrón JavaBeans. Es una forma del patrón
Builder [Gamma95]. En lugar de hacer el objeto deseado directamente, el cliente llama a un
constructor (o fábrica estática) con todos los parámetros requeridos y obtiene un objeto builder.
Luego, el cliente llama a métodos similares a setters en el objeto builder para establecer cada
parámetro opcional de interés. Finalmente, el cliente llama a un método build sin parámetros
para generar el objeto, que típicamente es inmutable. El builder es típicamente una clase
miembro estática (Elemento 24) de la clase que construye. Así es como se ve en la práctica:

```java
// Patrón Builder
public class NutritionFacts {
    private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;

    public static class Builder {
        // Parámetros obligatorios
        private final int servingSize;
        private final int servings;

        // Parámetros opcionales - inicializados a valores predeterminados
        private int calories = 0;
        private int fat = 0;
        private int sodium = 0;
        private int carbohydrate = 0;

        public Builder(int servingSize, int servings) {
            this.servingSize = servingSize;
            this.servings = servings;
        }

        public Builder calories(int val) { calories = val; return this; }
        public Builder fat(int val) { fat = val; return this; }
        public Builder sodium(int val) { sodium = val; return this; }
        public Builder carbohydrate(int val) { carbohydrate = val; return this; }

        public NutritionFacts build() {
            return new NutritionFacts(this);
        }
    }

    private NutritionFacts(Builder builder) {
        servingSize = builder.servingSize;
        servings = builder.servings;
        calories = builder.calories;
        fat = builder.fat;
        sodium = builder.sodium;
        carbohydrate = builder.carbohydrate;
    }
}
```

La clase `NutritionFacts` es inmutable, y todos los valores predeterminados de los parámetros 
están en un solo lugar. Los métodos setter del builder devuelven el propio builder para que 
las invocaciones se puedan encadenar, lo que resulta en una API fluida. Así es como se ve el 
código del cliente:

```java
NutritionFacts cocaCola = new NutritionFacts.Builder(240, 8)
    .calories(100).sodium(35).carbohydrate(27).build();
```

Este código de cliente es fácil de escribir y, lo que es más importante, fácil de leer. El patrón
Builder simula parámetros opcionales nombrados como se encuentran en Python y Scala.

Las comprobaciones de validez se omitieron por brevedad. Para detectar parámetros inválidos lo
antes posible, verifica la validez de los parámetros en el constructor del builder y en sus métodos.
Comprueba las invariantes que involucran múltiples parámetros en el constructor invocado por el
método build. Para asegurar estas invariantes contra ataques, realiza las comprobaciones en los
campos del objeto después de copiar los parámetros del builder (Elemento 50). Si una comprobación
falla, lanza una IllegalArgumentException (Elemento 72) cuyo mensaje detallado indica qué
parámetros son inválidos (Elemento 75).

El patrón Builder es adecuado para jerarquías de clases. Usa una jerarquía paralela de builders,
cada uno anidado en la clase correspondiente. Las clases abstractas tienen builders abstractos; las
clases concretas tienen builders concretos. Por ejemplo, considera una clase abstracta en la raíz de
una jerarquía que representa varios tipos de pizza

```java
// Patrón Builder para jerarquías de clases
public abstract class Pizza {
    public enum Topping { HAM, MUSHROOM, ONION, PEPPER, SAUSAGE }
    final Set<Topping> toppings;

    abstract static class Builder<T extends Builder<T>> {
        EnumSet<Topping> toppings = EnumSet.noneOf(Topping.class);

        public T addTopping(Topping topping) {
            toppings.add(Objects.requireNonNull(topping));
            return self();
        }

        abstract Pizza build();

        // Las subclases deben sobrescribir este método para devolver "this"
        protected abstract T self();
    }

    Pizza(Builder<?> builder) {
        toppings = builder.toppings.clone(); // Ver Elemento 50
    }
}
```
Nótese que `Pizza.Builder` es un tipo genérico con un parámetro de tipo recursivo 
(Elemento 30). Esto, junto con el método `self` abstracto, permite que el encadenamiento de 
métodos funcione correctamente en subclases, sin la necesidad de casts. Este método para 
compensar el hecho de que Java carece de un tipo self se conoce como el idiomático self-type 
simulado.


Aquí hay dos subclases concretas de `Pizza`, una de las cuales representa una pizza 
estilo Nueva York estándar, y la otra un calzone. La primera tiene un parámetro de tamaño 
requerido, mientras que la segunda te permite especificar si la salsa debe estar dentro o fuera
         
```java
public class NyPizza extends Pizza {
    public enum Size { SMALL, MEDIUM, LARGE }
    private final Size size;

    public static class Builder extends Pizza.Builder<Builder> {
        private final Size size;

        public Builder(Size size) {
            this.size = Objects.requireNonNull(size);
        }

        @Override 
        public NyPizza build() {
            return new NyPizza(this);
        }

        @Override 
        protected Builder self() { 
            return this; 
        }
    }

    private NyPizza(Builder builder) {
        super(builder);
        size = builder.size;
    }
}

public class Calzone extends Pizza {
    private final boolean sauceInside;

    public static class Builder extends Pizza.Builder<Builder> {
        private boolean sauceInside = false; // Por defecto

        public Builder sauceInside() {
            sauceInside = true;
            return this;
        }

        @Override 
        public Calzone build() {
            return new Calzone(this);
        }

        @Override 
        protected Builder self() { 
            return this; 
        }
    }

    private Calzone(Builder builder) {
        super(builder);
        sauceInside = builder.sauceInside;
    }
}
```
Nótese que el método `build` en el builder de cada subclase está declarado para devolver la 
subclase correcta: el método `build` de `NyPizza.Builder` devuelve `NyPizza`, mientras que 
el de `Calzone.Builder` devuelve `Calzone`. Esta técnica, en la que un método de subclase 
se declara para devolver un subtipo del tipo de retorno declarado en la superclase, se conoce 
como tipificación de retorno covariante. Permite a los clientes usar estos builders sin la 
necesidad de realizar casting.

El código del cliente para estos "builders jerárquicos" es esencialmente idéntico al código 
para el simple builder de `NutritionFacts`. El siguiente código de ejemplo del cliente asume 
importaciones estáticas de constantes enum por brevedad:

```java
NyPizza pizza = new NyPizza.Builder(SMALL)
    .addTopping(SAUSAGE).addTopping(ONION).build();

Calzone calzone = new Calzone.Builder()
    .addTopping(HAM).sauceInside().build();
```

Una ventaja menor de los builders sobre los constructores es que los builders pueden tener 
múltiples parámetros varargs, ya que cada parámetro se especifica en su propio método. 
Alternativamente, los builders pueden agregar los parámetros pasados en múltiples llamadas a 
un método en un solo campo, como se demostró en el método `addTopping` anteriormente.

El patrón Builder es bastante flexible. Un solo builder puede utilizarse repetidamente 
para construir múltiples objetos. Los parámetros del builder pueden ajustarse entre 
invocaciones del método `build` para variar los objetos que se crean. Un builder puede 
rellenar automáticamente algunos campos al crear el objeto, como un número de serie que 
aumenta cada vez que se crea un objeto.

El patrón Builder también tiene desventajas. Para crear un objeto, primero debes crear su 
builder. Aunque el costo de crear este builder probablemente no sea notable en la práctica, 
podría ser un problema en situaciones críticas de rendimiento. Además, el patrón Builder es 
más verboso que el patrón de constructor telescópico, por lo que solo debe usarse si hay 
suficientes parámetros para que valga la pena, digamos cuatro o más. Pero ten en cuenta que 
puedes querer agregar más parámetros en el futuro. Pero si comienzas con constructores o 
fábricas estáticas y cambias a un builder cuando la clase evoluciona al punto en que el número 
de parámetros se sale de control, los constructores o fábricas estáticas obsoletos resaltarán 
como un pulgar dolorido. Por lo tanto, a menudo es mejor comenzar con un builder desde el 
principio.

En resumen, el patrón Builder es una buena elección al diseñar clases cuyos constructores 
o fábricas estáticas tendrían más que un puñado de parámetros, especialmente si muchos de 
los parámetros son opcionales o del mismo tipo. El código del cliente es mucho más fácil de 
leer y escribir con builders que con constructores telescópicos, y los builders son mucho más 
seguros que los JavaBeans.

## Hacer cumplir la propiedad singleton con un constructor privado o un tipo enum
> SINGLETON
>
Un singleton es simplemente una clase que se instancia exactamente una vez [Gamma95]. Los singletons suelen representar ya sea un objeto sin estado, como una función (Elemento 24), o un componente del sistema que es intrínsecamente único. Hacer de una clase un singleton puede dificultar la prueba de sus clientes porque es imposible sustituir una implementación simulada (mock) para un singleton a menos que implemente una interfaz que sirva como su tipo.

Hay dos formas comunes de implementar singletons. Ambas se basan en mantener el constructor privado y exportar un miembro público estático para proporcionar acceso a la única instancia. En un enfoque, el miembro es un campo final:

```java
// Singleton con campo público final
public class Elvis {
    public static final Elvis INSTANCE = new Elvis();

    private Elvis() { 
        // Constructor privado
    }

    public void leaveTheBuilding() { 
        // Método de ejemplo
    }
}
```

La falta de un constructor público o protegido garantiza un universo "monoelvístico": exactamente una instancia de Elvis existirá una vez que la clase Elvis se haya inicializado, ni más ni menos. Nada de lo que haga un cliente puede cambiar esto, con una advertencia: un cliente con privilegios puede invocar el constructor privado reflexivamente (Elemento 65) con la ayuda del método `AccessibleObject.setAccessible`. Si necesitas defenderte contra este ataque, modifica el constructor para que lance una excepción si se le pide crear una segunda instancia.

En el segundo enfoque para implementar singletons, el miembro público es un método de fábrica estático:

```java
// Singleton con fábrica estática
public class Elvis {
    private static final Elvis INSTANCE = new Elvis();

    private Elvis() { 
        // Constructor privado
    }

    public static Elvis getInstance() { 
        return INSTANCE; 
    }

    public void leaveTheBuilding() { 
        // Método de ejemplo
    }
}
```


Todas las llamadas a `Elvis.getInstance` devuelven la misma referencia de objeto, y nunca se creará otra instancia de `Elvis` (con la misma advertencia mencionada anteriormente).


La principal ventaja del enfoque del campo público es que la API deja claro que la clase es un singleton: el campo público estático es final, por lo que siempre contendrá la misma referencia de objeto. La segunda ventaja es que es más simple.

Una ventaja del enfoque de la fábrica estática es que te brinda la flexibilidad de cambiar de opinión sobre si la clase es un singleton sin cambiar su API. El método de fábrica devuelve la única instancia, pero podría modificarse para devolver, por ejemplo, una instancia separada para cada hilo que lo invoque. Una segunda ventaja es que puedes escribir una fábrica de singleton genérica si tu aplicación lo requiere (Elemento 30). Una ventaja final de usar una fábrica estática es que una referencia de método se puede utilizar como proveedor, por ejemplo, `Elvis::instance` es un `Supplier<Elvis>`. A menos que una de estas ventajas sea relevante, el enfoque del campo público es preferible.

Para hacer que una clase singleton que utilice cualquiera de estos enfoques sea serializable (Capítulo 12), no es suficiente simplemente agregar `implements Serializable` a su declaración. Para mantener la garantía de singleton, declara todos los campos de instancia como transitorios y proporciona un método `readResolve` (Elemento 89). De lo contrario, cada vez que se deserializa una instancia serializada, se creará una nueva instancia, lo que, en el caso de nuestro ejemplo, podría llevar a avistamientos falsos de Elvis. Para evitar que esto ocurra, agrega este método `readResolve` a la clase `Elvis`:

```java
// Método readResolve para preservar la propiedad de singleton
private Object readResolve() {
    // Devolver el único Elvis real y dejar que el recolector de basura
    // se encargue del imitador de Elvis.
    return INSTANCE;
}
```

Una tercera forma de implementar un singleton es declarar un enum de un solo elemento:

```java
public enum ElvisV3 {
    INSTANCE;
    public void leaveTheBuilding() {
        // Método de ejemplo
    }
}
```
Este enfoque es similar al del campo público, pero más conciso, proporciona la maquinaria de serialización de forma gratuita y garantiza de manera sólida que no haya instanciaciones múltiples, incluso frente a sofisticados ataques de serialización o reflexión. Aunque este enfoque puede parecer un tanto antinatural, un tipo de enum de un solo elemento suele ser la mejor manera de implementar un singleton. Ten en cuenta que no puedes utilizar este enfoque si tu singleton debe extender una superclase que no sea `Enum` (aunque puedes declarar un enum para implementar interfaces).


otro ejemplo ejemplo, MySingletonEnum actúa como un singleton porque tiene solo una instancia, que es INSTANCE. Al acceder a MySingletonEnum.INSTANCE, obtienes la única instancia disponible. Además, implementa la interfaz MyInterface y proporciona una implementación concreta de myMethod. La clave aquí es que la instancia única es garantizada por la naturaleza de los enums en Java.

```java
// Interfaz que define el comportamiento común
interface MyInterface {
    void myMethod();
}

// Enum singleton que implementa la interfaz
public enum MySingletonEnum implements MyInterface {
    INSTANCE;

    @Override
    public void myMethod() {
        System.out.println("Implementación de myMethod en el enum singleton");
    }
}

// Clase de ejemplo que utiliza el enum singleton
public class Main {
    public static void main(String[] args) {
        // Acceder a la instancia única del enum singleton
        MySingletonEnum singleton = MySingletonEnum.INSTANCE;

        // Utilizar el método del singleton
        singleton.myMethod();
    }
}

```


## Hacer cumplir la no instanciación con un constructor privado

En ocasiones, querrás escribir una clase que sea simplemente un conjunto de métodos y campos estáticos. Estas clases han adquirido mala reputación porque algunas personas abusan de ellas para evitar pensar en términos de objetos, pero tienen usos válidos. Pueden utilizarse para agrupar métodos relacionados en valores primitivos o matrices, al estilo de `java.lang.Math` o `java.util.Arrays`. También pueden ser usadas para agrupar métodos estáticos, incluyendo fábricas (Item 1), para objetos que implementan alguna interfaz, al estilo de `java.util.Collections`. (A partir de Java 8, también puedes colocar esos métodos en la interfaz, siempre y cuando tengas permiso para modificarla). Por último, estas clases pueden ser utilizadas para agrupar métodos en una clase final, ya que no se pueden colocar en una subclase. Estas clases de utilidad no fueron diseñadas para ser instanciadas: una instancia sería sin sentido. Sin embargo, en ausencia de constructores explícitos, el compilador proporciona un constructor público sin parámetros por defecto. Para un usuario, este constructor no se distingue de cualquier otro. No es raro ver clases inadvertidamente instanciables en APIs publicadas.

Intentar forzar la no instanciabilidad haciendo que una clase sea abstracta no funciona. La clase puede ser subclasificada y la subclase instanciada. Además, induce al usuario a pensar que la clase fue diseñada para la herencia (Item 19). Sin embargo, hay un sencillo modismo para garantizar la no instanciabilidad. Un constructor por defecto solo se genera si una clase no contiene constructores explícitos, por lo que una clase puede hacerse no instanciable incluyendo un constructor privado:


```java
// Noninstantiable utility class
public class UtilityClass {
    // Suppress default constructor for noninstantiability
    private UtilityClass() {
        throw new AssertionError();
    }
    ... // Remainder omitted
}
```

Debido a que el constructor explícito es privado, es inaccesible fuera de la clase.
Si bien no es estrictamente necesario, el AssertionError proporciona una garantía en caso de que el constructor se invoque accidentalmente desde dentro de la clase. Asegura que la clase nunca será instanciada bajo ninguna circunstancia. Este modismo es ligeramente contra intuitivo porque el constructor se proporciona expresamente para que no pueda ser invocado. Por lo tanto, es recomendable incluir un comentario, como se mostró anteriormente.
Como efecto secundario, este modismo también evita que la clase sea subclasificada. Todos los constructores deben invocar explícita o implícitamente un constructor de la superclase, y una subclase no tendría ningún constructor de superclase accesible para invocar.

## Prefiere la inyección de dependencias a la codificación directa de recursos
Muchas clases dependen de uno o más recursos subyacentes. Por ejemplo, un corrector ortográfico depende de un diccionario. No es raro ver que estas clases se implementen como clases de utilidad estáticas (Item 4):


```java
// Uso inapropiado de utilidad estática: inflexible y no testeable
public class SpellChecker {
private static final Lexicon dictionary = ...;

    private SpellChecker() {} // No instanciable
    
    public static boolean isValid(String word) { ... }
    
    public static List<String> suggestions(String typo) { ... }
}
```

Igualmente, no es raro ver que se implementen como singletons (Item 3):

```java
// Uso inapropiado de singleton: inflexible y no testeable
public class SpellChecker {
    private final Lexicon dictionary = ...;
    
    private SpellChecker(...) {}
    
    public static final SpellChecker INSTANCE = new SpellChecker(...);
    
    public boolean isValid(String word) { ... }
    
    public List<String> suggestions(String typo) { ... }
}
```

Ninguno de estos enfoques es satisfactorio, ya que asumen que hay solo un diccionario que vale la pena usar. En la práctica, cada idioma tiene su propio diccionario, y se utilizan diccionarios especiales para vocabularios específicos. Además, puede ser deseable utilizar un diccionario especial para pruebas. Es ingenuo pensar que un solo diccionario será suficiente para siempre.

Podrías intentar que SpellChecker admita múltiples diccionarios haciendo que el campo del diccionario no sea final y agregando un método para cambiar el diccionario en un corrector ortográfico existente, pero esto sería incómodo, propenso a errores e ineficaz en un entorno concurrente. Las clases de utilidad estática y los singletons no son apropiados para clases cuyo comportamiento está parametrizado por un recurso subyacente.

Lo que se requiere es la capacidad de admitir múltiples instancias de la clase (en nuestro ejemplo, SpellChecker), cada una de las cuales utiliza el recurso deseado por el cliente (en nuestro ejemplo, el diccionario). Un patrón simple que satisface este requisito es pasar el recurso al constructor al crear una nueva instancia. Esto es una forma de inyección de dependencias: el diccionario es una dependencia del corrector ortográfico y se inyecta en el corrector ortográfico cuando se crea.
```java
// La inyección de dependencias proporciona flexibilidad y testeabilidad
public class SpellChecker {
    private final Lexicon dictionary;

    public SpellChecker(Lexicon dictionary) {
        this.dictionary = Objects.requireNonNull(dictionary);
    }

    public boolean isValid(String word) { ... }

    public List<String> suggestions(String typo) { ... }
}
```
El patrón de inyección de dependencias es tan simple que muchos programadores lo utilizan durante años sin saber que tiene un nombre. Mientras que nuestro ejemplo de corrector ortográfico tenía solo un recurso (el diccionario), la inyección de dependencias funciona con un número arbitrario de recursos y gráficos de dependencias arbitrarios. Preserva la inmutabilidad (Item 17), por lo que varios clientes pueden compartir objetos dependientes (si los clientes desean los mismos recursos subyacentes). La inyección de dependencias es igualmente aplicable a constructores, fábricas estáticas (Item 1) y constructores (Item 2).

Una variante útil del patrón es pasar un factory de recurso al constructor. Un factory es un objeto que se puede llamar repetidamente para crear instancias de un tipo. Tales factories encarnan el patrón Factory Method [Gamma95]. La interfaz Supplier<T>, introducida en Java 8, es perfecta para representar factories. Los métodos que toman un Supplier<T> como entrada deben limitar típicamente el parámetro de tipo del factory utilizando un tipo de comodín delimitado (Item 31) para permitir que el cliente pase un factory que cree cualquier subtipo de un tipo especificado. Por ejemplo, aquí hay un método que crea un mosaico utilizando un factory proporcionado por el cliente para producir cada azulejo:

```java
Mosaic create(Supplier<? extends Tile> tileFactory) { ... }
```
Aunque la inyección de dependencias mejora significativamente la flexibilidad y la testeabilidad, puede ensuciar proyectos grandes, que típicamente contienen miles de dependencias. Esta confusión puede eliminarse casi por completo utilizando un framework de inyección de dependencias, como Dagger [Dagger], Guice [Guice] o Spring [Spring]. El uso de estos frameworks está más allá del alcance de este libro, pero cabe destacar que las APIs diseñadas para la inyección de dependencias manual se adaptan fácilmente para su uso con estos frameworks.

En resumen, no uses un singleton o una clase de utilidad estática para implementar una clase que dependa de uno o más recursos subyacentes cuyo comportamiento afecte al de la clase, y no hagas que la clase cree estos recursos directamente. En su lugar, pasa los recursos o factories para crearlos al constructor (o fábrica estática o constructor). Esta práctica, conocida como inyección de dependencias, mejorará significativamente la flexibilidad, reutilización y testeabilidad de una clase.

## Evite crear objetos innecesarios

A menudo es apropiado reutilizar un solo objeto en lugar de crear uno nuevo cada vez que se necesita. La reutilización puede ser más rápida y elegante. Un objeto siempre puede ser reutilizado si es inmutable (Ítem 17).
Como un ejemplo extremo de lo que no se debe hacer, considera esta declaración:

```java
String s = new String("bikini"); // DON'T DO THIS!
```
La declaración crea una nueva instancia de String cada vez que se ejecuta, y ninguna de esas creaciones de objetos es necesaria. El argumento del constructor de String ("bikini") es en sí mismo una instancia de String, funcionalmente idéntica a todos los objetos creados por el constructor. Si este uso ocurre en un bucle o en un método invocado con frecuencia, se pueden crear millones de instancias de String innecesariamente.
La versión mejorada es simplemente la siguiente:

```java
 String s = "bikini";
```

Esta versión utiliza una única instancia de String en lugar de crear una nueva cada vez que se ejecuta. Además, se garantiza que el objeto será reutilizado por cualquier otro código que se ejecute en la misma máquina virtual y que contenga el mismo literal de cadena [JLS, 3.10.5].
A menudo puedes evitar la creación de objetos innecesarios utilizando métodos de fábrica estáticos (Ítem 1) en lugar de constructores en clases inmutables que los proporcionan.
Por ejemplo, el método de fábrica Boolean.valueOf(String) es preferible al constructor Boolean(String), que fue deprecado en Java 9. El constructor debe crear un nuevo objeto cada vez que se llama, mientras que el método de fábrica nunca está obligado a hacerlo y en la práctica no lo hará. Además de reutilizar objetos inmutables, también puedes reutilizar objetos mutables si sabes que no serán modificados.
Algunas creaciones de objetos son mucho más costosas que otras. Si necesitas dicho "objeto costoso" repetidamente, puede ser aconsejable almacenarlo en caché para su reutilización. Desafortunadamente, no siempre es obvio cuando estás creando dicho objeto.
Supongamos que quieres escribir un método para determinar si una cadena es un numeral romano válido. Aquí está la forma más sencilla de hacerlo usando una expresión regular:


```java
// Performance can be greatly improved!
static boolean isRomanNumeral(String s) {
     return s.matches("^(?=.)M*(C[MD]|D?C{0,3})(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");
}
```
El problema con esta implementación es que depende del método String.matches. Si bien String.matches es la forma más sencilla de verificar si una cadena coincide con una expresión regular, no es adecuado para su uso repetido en situaciones críticas de rendimiento. El problema es que internamente crea una instancia de Pattern para la expresión regular y la utiliza solo una vez, después de lo cual se vuelve elegible para la recolección de basura. Crear una instancia de Pattern es costoso porque requiere compilar la expresión regular en una máquina de estados finitos.
Para mejorar el rendimiento, compila explícitamente la expresión regular en una instancia de Pattern (que es inmutable) como parte de la inicialización de la clase, cáchela y reutiliza la misma instancia para cada invocación del método isRomanNumeral:

```java
// Reusing expensive object for improved performance
public class RomanNumerals {
   private static final Pattern ROMAN = Pattern.compile("^(?=.)M*(C[MD]|D?C{0,3})(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");
   static boolean isRomanNumeral(String s) {
      return ROMAN.matcher(s).matches();
   }
}
```
La versión mejorada de `isRomanNumeral` proporciona ganancias significativas de rendimiento si se invoca con frecuencia. En mi máquina, la versión original tarda 1.1 µs en una cadena de entrada de 8 caracteres, mientras que la versión mejorada tarda 0.17 µs, lo que es 6.5 veces más rápido. No solo mejora el rendimiento, sino que también se podría argumentar que mejora la claridad.

Crear un campo estático final para la instancia de `Pattern`, que de otro modo sería invisible, nos permite darle un nombre, que es mucho más legible que la expresión regular en sí.

Si la clase que contiene la versión mejorada del método `isRomanNumeral` se inicializa pero el método nunca se invoca, el campo `ROMAN` se inicializará innecesariamente. Sería posible eliminar la inicialización mediante la inicialización perezosa del campo (Elemento 83) la primera vez que se invoque el método `isRomanNumeral`, pero esto no se recomienda. Como suele ser el caso con la inicialización perezosa, complicaría la implementación sin una mejora de rendimiento mensurable (Elemento 67).

Cuando un objeto es inmutable, es obvio que se puede reutilizar de manera segura, pero hay otras situaciones donde es mucho menos obvio, incluso contraintuitivo. Considera el caso de los adaptadores [Gamma95], también conocidos como vistas. Un adaptador es un objeto que delega a un objeto de respaldo, proporcionando una interfaz alternativa. Dado que un adaptador no tiene estado más allá del objeto de respaldo, no hay necesidad de crear más de una instancia de un adaptador dado para un objeto dado.

Por ejemplo, el método `keySet` de la interfaz `Map` devuelve una vista de tipo `Set` del objeto `Map`, que consiste en todas las claves en el mapa. Ingenuamente, parecería que cada llamada a `keySet` tendría que crear una nueva instancia de `Set`, pero cada llamada a `keySet` en un objeto `Map` dado puede devolver la misma instancia de `Set`. Aunque la instancia de `Set` devuelta suele ser mutable, todos los objetos devueltos son funcionalmente idénticos: cuando uno de los objetos devueltos cambia, también lo hacen los demás, porque todos están respaldados por la misma instancia de `Map`. Aunque crear múltiples instancias del objeto de vista `keySet` generalmente no causa problemas, es innecesario y no ofrece beneficios.

Otra manera de crear objetos innecesarios es mediante el autoboxing, que permite al programador mezclar tipos primitivos y tipos primitivos en caja, realizando el envasado y desenvasado automáticamente según sea necesario. El autoboxing difumina pero no borra la distinción entre tipos primitivos y tipos primitivos en caja. Hay distinciones semánticas sutiles y diferencias de rendimiento no tan sutiles (Elemento 61). Considera el siguiente método, que calcula la suma de todos los valores enteros positivos. Para hacer esto, el programa tiene que usar aritmética `long` porque un `int` no es lo suficientemente grande para contener la suma de todos los valores enteros positivos.

```java
// Hideously slow! Can you spot the object creation?
private static long sum() {
  Long sum = 0L;
  for (long i = 0; i <= Integer.MAX_VALUE; i++)
    sum += i;
    return sum;
  }
}
```

El tipo Long es un objeto, y al usarlo en un contexto donde se espera un tipo primitivo long, se realiza una operación llamada autoboxing, que convierte automáticamente el tipo primitivo en su correspondiente tipo de objeto (Long en este caso). La creación repetida de objetos Long en cada iteración del bucle puede ser ineficiente, especialmente en bucles grandes.

Para mejorar la eficiencia, puedes utilizar el tipo primitivo long directamente en lugar de Long. Aquí está el código optimizado:

```java
private static long sum() {
  long sum = 0L;
  for (long i = 0; i <= Integer.MAX_VALUE; i++)
    sum += i;
  return sum;
}
```
Este programa obtiene la respuesta correcta, pero es mucho más lento de lo que debería debido a un error tipográfico de un solo carácter. La variable `sum` está declarada como un `Long` en lugar de un `long`, lo que significa que el programa construye alrededor de 2^31 instancias innecesarias de `Long` (aproximadamente una por cada vez que se suma el `long i` al `Long sum`). Cambiar la declaración de `sum` de `Long` a `long` reduce el tiempo de ejecución de 6.3 segundos a 0.59 segundos en mi máquina. La lección es clara: prefiera los primitivos a los primitivos en caja y tenga cuidado con el autoboxing no intencional.

Este elemento no debe malinterpretarse para implicar que la creación de objetos es cara y debe evitarse. Por el contrario, la creación y recuperación de objetos pequeños cuyos constructores hacen poco trabajo explícito son baratas, especialmente en las implementaciones modernas de JVM. Crear objetos adicionales para mejorar la claridad, simplicidad o potencia de un programa generalmente es algo positivo.

En cambio, evitar la creación de objetos mediante el mantenimiento de tu propio grupo de objetos es una mala idea a menos que los objetos en el grupo sean extremadamente pesados. El ejemplo clásico de un objeto que justifica un grupo de objetos es una conexión a la base de datos. El costo de establecer la conexión es lo suficientemente alto como para tener sentido reutilizar estos objetos. Sin embargo, en general, mantener tus propios grupos de objetos ensucia tu código, aumenta la huella de memoria y perjudica el rendimiento. Las implementaciones modernas de la JVM tienen recolectores de basura altamente optimizados que superan fácilmente a dichos grupos de objetos en objetos livianos.

El contrapunto a este ítem es el ítem 50 sobre copias defensivas. Este ítem dice: "No crees un nuevo objeto cuando deberías reutilizar uno existente", mientras que el ítem 50 dice: "No reutilices un objeto existente cuando deberías crear uno nuevo". Ten en cuenta que la penalización por reutilizar un objeto cuando se requiere copia defensiva es mucho mayor que la penalización por crear innecesariamente un objeto duplicado. No hacer copias defensivas cuando sea necesario puede conducir a errores insidiosos y agujeros de seguridad; crear objetos innecesariamente solo afecta al estilo y al rendimiento.

## Elimine las referencias a objetos obsoletos
Si cambiaste de un lenguaje con gestión manual de memoria, como C o C++, a un lenguaje con recolector de basura, como Java, tu trabajo como programador se simplificó considerablemente debido a que tus objetos se reclaman automáticamente cuando terminas de usarlos. Parece casi como magia la primera vez que lo experimentas. Puede dar la impresión de que no tienes que preocuparte por la gestión de memoria, pero esto no es del todo cierto.


```java
public class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }
    public void push(Object e) {
        ensureCapacity();
        elements[size++] = e;
    }
    public Object pop() {
        if (size == 0)
            throw new EmptyStackException();
        return elements[--size];
    }

    private void ensureCapacity() {
        if (elements.length == size)
            elements = Arrays.copyOf(elements, 2 * size + 1);
    }

    public int getSize() {
        return this.size;
    }
}
```

No hay nada evidentemente incorrecto en este programa (pero consulta el Ítem 29 para una versión genérica). Podrías probarlo exhaustivamente y pasaría cada prueba con colores brillantes, pero hay un problema acechando. Hablando de manera general, el programa tiene una "fuga de memoria", que puede manifestarse silenciosamente como un rendimiento reducido debido a un aumento en la actividad del recolector de basura o un mayor uso de memoria. En casos extremos, estas fugas de memoria pueden causar paginación de disco e incluso fallos en el programa con un OutOfMemoryError, aunque tales fallos son relativamente raros.

Entonces, ¿dónde está la fuga de memoria? Si una pila crece y luego disminuye, los objetos que se sacaron de la pila no serán recolectados por el recolector de basura, incluso si el programa que utiliza la pila no tiene más referencias a ellos. Esto se debe a que la pila mantiene referencias obsoletas a estos objetos. Una referencia obsoleta es simplemente una referencia que nunca se desreferenciará de nuevo. En este caso, cualquier referencia fuera de la "porción activa" del arreglo de elementos es obsoleta. La porción activa consiste en los elementos cuyo índice es menor que el tamaño.

Las fugas de memoria en lenguajes con recolección de basura (más correctamente conocidas como retenciones no intencionales de objetos) son insidiosas. Si se retiene una referencia de objeto sin intención, no solo se excluye ese objeto de la recolección de basura, sino también cualquier objeto referenciado por ese objeto, y así sucesivamente. Incluso si solo se retienen unas pocas referencias de objetos sin intención, muchos objetos pueden quedar fuera de la recolección de basura, con efectos potencialmente grandes en el rendimiento.

La solución para este tipo de problema es simple: anula las referencias una vez que se vuelven obsoletas. En el caso de nuestra clase Stack, la referencia a un elemento se vuelve obsoleta tan pronto como se saca de la pila. La versión corregida del método pop se ve así:

```java
    public Object pop() {
        if (size == 0)
            throw new EmptyStackException();
        Object result = elements[--size];
        elements[size] = null;
        return result;
    }
```
Un beneficio adicional de anular las referencias obsoletas es que, si posteriormente se desreferencian por error, el programa fallará de inmediato con una NullPointerException, en lugar de realizar silenciosamente la acción incorrecta. Siempre es beneficioso detectar errores de programación lo más rápido posible.

Cuando los programadores son picados por este problema por primera vez, pueden reaccionar exageradamente anulando cada referencia de objeto tan pronto como el programa haya terminado de usarla. Esto ni es necesario ni deseable; simplemente ensucia innecesariamente el programa. Anular referencias de objetos debería ser la excepción más que la norma.

La mejor manera de eliminar una referencia obsoleta es permitir que la variable que contenía la referencia caiga fuera de alcance. Esto ocurre naturalmente si defines cada variable en el alcance más estrecho posible 


Entonces, ¿cuándo deberías anular una referencia? ¿Qué aspecto de la clase Stack la hace propensa a fugas de memoria? En pocas palabras, gestiona su propia memoria. El conjunto de almacenamiento consiste en los elementos del arreglo de elementos (las celdas de referencia de objetos, no los objetos en sí). Los elementos en la porción activa del arreglo (como se definió anteriormente) están asignados, y aquellos en el resto del arreglo están libres. El recolector de basura no tiene forma de saber esto; para el recolector de basura, todas las referencias de objetos en el arreglo de elementos son igualmente válidas. Solo el programador sabe que la porción inactiva del arreglo no es importante. El programador comunica efectivamente este hecho al recolector de basura anulando manualmente los elementos del arreglo tan pronto como se convierten en parte de la porción inactiva.

En general, siempre que una clase gestione su propia memoria, el programador debe estar atento a las fugas de memoria. Siempre que se libere un elemento, cualquier referencia de objeto contenida en el elemento debería anularse.

Otra fuente común de fugas de memoria son las cachés. Una vez que colocas una referencia de objeto en una caché, es fácil olvidar que está allí y dejarla en la caché mucho después de que sea irrelevante. Hay varias soluciones para este problema. Si tienes la suerte de implementar una caché para la cual una entrada es relevante exactamente mientras haya referencias a su clave fuera de la caché, representa la caché como un WeakHashMap; las entradas se eliminarán automáticamente después de volverse obsoletas. Recuerda que WeakHashMap es útil solo si la vida útil deseada de las entradas de la caché está determinada por referencias externas a la clave, no al valor.

Más comúnmente, la vida útil útil de una entrada de caché está menos definida, con entradas que se vuelven menos valiosas con el tiempo. En estas circunstancias, la caché debería limpiarse ocasionalmente de entradas que han caído en desuso. Esto se puede hacer mediante un hilo en segundo plano (quizás un ScheduledThreadPoolExecutor) o como un efecto secundario al agregar nuevas entradas a la caché. La clase LinkedHashMap facilita este último enfoque con su método removeEldestEntry. Para cachés más sofisticadas, es posible que necesites usar java.lang.ref directamente.

Una tercera fuente común de fugas de memoria son los listeners y otros callbacks. Si implementas una API donde los clientes registran callbacks pero no los cancelan explícitamente, se acumularán a menos que tomes alguna medida. Una forma de asegurarte de que los callbacks se recojan de manera pronta es almacenar solo referencias débiles a ellos, por ejemplo, almacenándolos solo como claves en un WeakHashMap.

Dado que las fugas de memoria generalmente no se manifiestan como fallas obvias, pueden permanecer presentes en un sistema durante años. Típicamente, se descubren solo como resultado de una inspección cuidadosa del código o con la ayuda de una herramienta de depuración conocida como un perfilador de montón. Por lo tanto, es muy deseable aprender a anticipar problemas como este antes de que ocurran y evitar que sucedan.

## Evita finalizadores y limpiadores
Los finalizadores son impredecibles, a menudo peligrosos y generalmente innecesarios. Su uso puede causar un comportamiento errático, un rendimiento deficiente y problemas de portabilidad. Los finalizadores tienen algunos usos válidos, que cubriremos más adelante en este apartado, pero como regla general, deberías evitarlos. A partir de Java 9, los finalizadores han sido deprecados, pero aún están siendo utilizados por las bibliotecas de Java. El reemplazo de Java 9 para los finalizadores son los limpiadores. Los limpiadores son menos peligrosos que los finalizadores, pero aún son impredecibles, lentos y generalmente innecesarios.

Los programadores de C++ deben tener cuidado de no pensar en los finalizadores o limpiadores como el análogo en Java de los destructores de C++. En C++, los destructores son la forma normal de recuperar los recursos asociados con un objeto, un complemento necesario para los constructores. En Java, el recolector de basura reclama el almacenamiento asociado con un objeto cuando se vuelve inaccesible, sin requerir ningún esfuerzo especial por parte del programador. Los destructores de C++ también se utilizan para recuperar otros recursos que no son de memoria. En Java, un bloque try-with-resources o try-finally se utiliza para este propósito (Apartado 9).

Una limitación de los finalizadores y limpiadores es que no hay garantía de que se ejecuten de manera oportuna [JLS, 12.6]. Puede pasar un tiempo arbitrariamente largo entre el momento en que un objeto se vuelve inaccesible y el momento en que se ejecuta su finalizador o limpiador. Esto significa que nunca deberías hacer nada crítico en términos de tiempo en un finalizador o limpiador. Por ejemplo, es un error grave depender de un finalizador o limpiador para cerrar archivos porque los descriptores de archivos abiertos son un recurso limitado. Si se dejan muchos archivos abiertos como resultado de la tardanza del sistema en ejecutar los finalizadores o limpiadores, un programa puede fallar porque ya no puede abrir archivos.

La rapidez con la que se ejecutan los finalizadores y limpiadores es principalmente una función del algoritmo de recolección de basura, que varía ampliamente entre las implementaciones. El comportamiento de un programa que depende de la prontitud de la ejecución del finalizador o limpiador también puede variar. Es completamente posible que dicho programa se ejecute perfectamente en la JVM en la que lo pruebas y luego falle miserablemente en la que prefiera tu cliente más importante.

La finalización tardía no es solo un problema teórico. Proporcionar un finalizador para una clase puede retrasar arbitrariamente la reclamación de sus instancias. Un colega depuró una aplicación de GUI de larga duración que moría misteriosamente con un OutOfMemoryError. El análisis reveló que en el momento de su muerte, la aplicación tenía miles de objetos gráficos en su cola de finalización esperando ser finalizados y reclamados. Desafortunadamente, el hilo de finalización se estaba ejecutando con una prioridad más baja que otro hilo de aplicación, por lo que los objetos no se finalizaban al ritmo al que se volvían elegibles para la finalización.

La especificación del lenguaje no garantiza qué hilo ejecutará los finalizadores, por lo que no hay una manera portátil de prevenir este tipo de problema aparte de abstenerse de usar finalizadores. Los limpiadores son un poco mejores que los finalizadores en este aspecto porque los autores de clases tienen control sobre sus propios hilos de limpieza, pero los limpiadores aún se ejecutan en segundo plano, bajo el control del recolector de basura, por lo que no puede garantizarse una limpieza oportuna. No solo la especificación no garantiza que los finalizadores o limpiadores se ejecuten de manera oportuna; tampoco garantiza que se ejecuten en absoluto. Es completamente posible, e incluso probable, que un programa termine sin ejecutarlos en algunos objetos que ya no son alcanzables. Como consecuencia, nunca debes depender de un finalizador o limpiador para actualizar un estado persistente. Por ejemplo, depender de un finalizador o limpiador para liberar un bloqueo persistente en un recurso compartido como una base de datos es una buena manera de detener por completo tu sistema distribuido.

No te dejes seducir por los métodos System.gc y System.runFinalization. Pueden aumentar las probabilidades de que los finalizadores o limpiadores se ejecuten, pero no lo garantizan. Una vez se afirmó que dos métodos garantizaban esto: System.runFinalizersOnExit y su gemelo maligno, Runtime.runFinalizersOnExit. Estos métodos tienen fallas fatales y han sido deprecados durante décadas. Otro problema con los finalizadores es que se ignora una excepción no capturada lanzada durante la finalización, y la finalización de ese objeto termina. Las excepciones no capturadas pueden dejar otros objetos en un estado corrupto. Si otro hilo intenta usar un objeto corrupto, puede resultar en un comportamiento no determinístico arbitrario. Normalmente, una excepción no capturada terminará el hilo e imprimirá una traza de pila, pero no si ocurre en un finalizador; ni siquiera imprimirá una advertencia. Los limpiadores no tienen este problema porque una biblioteca que utiliza un limpiador tiene control sobre su hilo.

Hay una penalización de rendimiento severa por usar finalizadores y limpiadores. En mi máquina, el tiempo para crear un objeto AutoCloseable simple, cerrarlo usando try-with-resources y que el recolector de basura lo reclame es de aproximadamente 12 ns. Usar un finalizador en su lugar aumenta el tiempo a 550 ns. En otras palabras, es aproximadamente 50 veces más lento crear y destruir objetos con finalizadores. Esto se debe principalmente a que los finalizadores inhiben una recolección de basura eficiente. Los limpiadores son comparables en velocidad a los finalizadores si los usas para limpiar todas las instancias de la clase (alrededor de 500 ns por instancia en mi máquina), pero los limpiadores son mucho más rápidos si los usas solo como una red de seguridad, como se discute a continuación. En estas circunstancias, crear, limpiar y destruir un objeto toma alrededor de 66 ns en mi máquina, lo que significa que pagas un factor de cinco (no cincuenta) por el seguro de una red de seguridad si no lo usas.

Los finalizadores tienen un grave problema de seguridad: abren tu clase a ataques de finalizadores. La idea detrás de un ataque de finalizador es simple: si se lanza una excepción desde un constructor o sus equivalentes de serialización, los métodos readObject y readResolve (Capítulo 12), el finalizador de una subclase maliciosa puede ejecutarse en el objeto parcialmente construido que debería haber "muerto en la parra". Este finalizador puede registrar una referencia al objeto en un campo estático, evitando que sea recolectado por el recolector de basura. Una vez que el objeto malformado ha sido registrado, es un simple asunto invocar métodos arbitrarios en este objeto que nunca deberían haber sido permitidos existir en primer lugar. Lanzar una excepción desde un constructor debería ser suficiente para evitar que un objeto llegue a existir; en presencia de finalizadores, no lo es. Tales ataques pueden tener consecuencias graves. Las clases finales son inmunes a los ataques de finalizadores porque nadie puede escribir una subclase maliciosa de una clase final. Para proteger las clases no finales de los ataques de finalizadores, escribe un método final finalize que no haga nada.

Entonces, ¿qué deberías hacer en lugar de escribir un finalizador o limpiador para una clase cuyos objetos encapsulan recursos que requieren terminación, como archivos o hilos? Simplemente haz que tu clase implemente AutoCloseable y requiere que sus clientes invoquen el método close en cada instancia cuando ya no sea necesario, típicamente usando try-with-resources para garantizar la terminación incluso en caso de excepciones (Item 9). Un detalle que vale la pena mencionar es que la instancia debe llevar un registro de si ha sido cerrada: el método close debe registrar en un campo que el objeto ya no es válido, y otros métodos deben verificar este campo y lanzar una IllegalStateException si son llamados después de que el objeto haya sido cerrado.

Entonces, ¿para qué sirven los limpiadores y finalizadores, si es que sirven para algo? Tienen quizás dos usos legítimos. Uno es actuar como una red de seguridad en caso de que el propietario de un recurso descuide llamar a su método close. Aunque no hay garantía de que el limpiador o finalizador se ejecute de manera oportuna (o en absoluto), es mejor liberar el recurso tarde que nunca si el cliente falla en hacerlo. Si estás considerando escribir un finalizador de red de seguridad, piensa detenidamente si la protección vale la pena el costo. Algunas clases de biblioteca de Java, como FileInputStream, FileOutputStream, ThreadPoolExecutor y java.sql.Connection, tienen finalizadores que actúan como redes de seguridad.

Un segundo uso legítimo de los limpiadores concierne a los objetos con pares nativos. Un par nativo es un objeto nativo (no Java) al que un objeto normal delega mediante métodos nativos. Como un par nativo no es un objeto normal, el recolector de basura no lo conoce y no puede reclamarlo cuando su par de Java es reclamado. Un limpiador o finalizador puede ser un vehículo apropiado para esta tarea, asumiendo que el rendimiento es aceptable y que el par nativo no tiene recursos críticos. Si el rendimiento no es aceptable o el par nativo tiene recursos que deben ser reclamados de manera oportuna, la clase debería tener un método close, como se describió anteriormente.


Los limpiadores son un poco difíciles de usar. A continuación se muestra una clase Room simple que demuestra la funcionalidad. Supongamos que las habitaciones deben limpiarse antes de ser reclamadas. La clase Room implementa AutoCloseable; el hecho de que su red de seguridad de limpieza automática utilice un limpiador es simplemente un detalle de implementación. A diferencia de los finalizadores, los limpiadores no contaminan la API pública de una clase.

```java
// An autocloseable class using a cleaner as a safety net
public class Room implements AutoCloseable {
    private static final Cleaner cleaner = Cleaner.create();

    // Resource that requires cleaning. Must not refer to Room!
    private static class State implements Runnable {
        int numJunkPiles; // Number of junk piles in this room

        State(int numJunkPiles) {
            this.numJunkPiles = numJunkPiles;
        }

        // Invoked by close method or cleaner
        @Override
        public void run() {
            System.out.println("Cleaning room");
            numJunkPiles = 0;
        }
    }

    // The state of this room, shared with our cleanable
    private final State state;

    // Our cleanable. Cleans the room when it’s eligible for gc
    private final Cleaner.Cleanable cleanable;

    public Room(int numJunkPiles) {
        state = new State(numJunkPiles);
        cleanable = cleaner.register(this, state);
    }

    @Override
    public void close() {
        cleanable.clean();
    }
}

```

## Preferir try-with-resources a try-finally
Las bibliotecas de Java incluyen muchos recursos que deben cerrarse manualmente mediante la invocación de un método close. Ejemplos incluyen InputStream, OutputStream y java.sql.Connection. El cierre de recursos a menudo es pasado por alto por los clientes, con consecuencias previsiblemente desastrosas para el rendimiento. Aunque muchos de estos recursos utilizan finalizadores como red de seguridad, los finalizadores no funcionan muy bien (Item 8).

Históricamente, una declaración try-finally era la mejor manera de garantizar que un recurso se cerrara correctamente, incluso en caso de una excepción o un retorno.


```java
// try-finally - ¡Ya no es la mejor manera de cerrar recursos!
static String firstLineOfFile(String path) throws IOException {
    BufferedReader br = new BufferedReader(new FileReader(path));
    try {
        return br.readLine();
    } finally {
        br.close();
    }
}

// try-finally es feo cuando se usa con más de un recurso.
static void copy(String src, String dst) throws IOException {
    InputStream in = new FileInputStream(src);
    try {
        OutputStream out = new FileOutputStream(dst);
        try {
            byte[] buf = new byte[BUFFER_SIZE];
            int n;
            while ((n = in.read(buf)) >= 0)
                out.write(buf, 0, n);
        } finally {
            out.close();
        }
    } finally {
        in.close();
    }
}
```

Puede ser difícil de creer, pero incluso buenos programadores lo hicieron mal la mayor parte del tiempo. Para empezar, lo hice mal en la página 88 de Java Puzzlers [Bloch05], y nadie lo notó durante años. De hecho, dos tercios de los usos del método close en las bibliotecas de Java estaban mal en 2007.

Incluso el código correcto para cerrar recursos con declaraciones try-finally, como se ilustra en los dos ejemplos de código anteriores, tiene una deficiencia sutil. El código tanto en el bloque try como en el bloque finally es capaz de lanzar excepciones. Por ejemplo, en el método firstLineOfFile, la llamada a readLine podría lanzar una excepción debido a un fallo en el dispositivo físico subyacente, y la llamada a close también podría fallar por la misma razón. En estas circunstancias, la segunda excepción obliterate completamente la primera. No hay ningún registro de la primera excepción en el rastreo de la pila de excepciones, lo que puede complicar enormemente la depuración en sistemas reales, por lo general es la primera excepción la que desea ver para diagnosticar el problema. Aunque es posible escribir código para suprimir la segunda excepción en favor de la primera, prácticamente nadie lo hizo porque es demasiado verbose.

Todos estos problemas se resolvieron de una vez cuando Java 7 introdujo la declaración 'try-with-resources'. Para ser utilizable con esta construcción, un recurso debe implementar la interfaz AutoCloseable, que consiste en un único método close que devuelve void. Muchas clases e interfaces en las bibliotecas de Java y en bibliotecas de terceros ahora implementan o extienden AutoCloseable. Si escribes una clase que representa un recurso que debe cerrarse, tu clase también debería implementar AutoCloseable.


```java
// try-with-resources - ¡la mejor manera de cerrar recursos!
static String firstLineOfFile(String path) throws IOException {
    try (BufferedReader br = new BufferedReader(new FileReader(path))) {
        return br.readLine();
    }
}

// try-with-resources en múltiples recursos - corto y dulce
static void copy(String src, String dst) throws IOException {
    try (InputStream in = new FileInputStream(src);
         OutputStream out = new FileOutputStream(dst)) {
        byte[] buf = new byte[BUFFER_SIZE];
        int n;
        while ((n = in.read(buf)) >= 0) {
            out.write(buf, 0, n);
        }
    }
}

```
No solo las versiones try-with-resources son más cortas y legibles que las originales, sino que también proporcionan diagnósticos mucho mejores. Considera el método firstLineOfFile

El método firstLineOfFile. Si se lanzan excepciones tanto por la llamada readLine como por el cierre (invisible), la última excepción se suprime en favor de la primera. De hecho, varias excepciones pueden ser suprimidas para preservar la excepción que realmente desea ver. Estas excepciones suprimidas no son simplemente descartadas; se imprimen en la traza de pila con una notación que indica que fueron suprimidas. También puedes acceder a ellas programáticamente con el método getSuppressed, que se agregó a Throwable en Java 7.

Puedes poner cláusulas catch en las declaraciones try-with-resources, al igual que en las declaraciones try-finally regulares. Esto te permite manejar excepciones sin ensuciar tu código con otra capa de anidamiento. Como ejemplo un poco forzado, aquí tienes una versión de nuestro método firstLineOfFile que no lanza excepciones, sino que toma un valor predeterminado para devolver si no puede abrir el archivo o leer de él.

```java
// try-with-resources con una cláusula catch
static String firstLineOfFile(String path, String defaultVal) {
    try (BufferedReader br = new BufferedReader(new FileReader(path))) {
        return br.readLine();
    } catch (IOException e) {
        return defaultVal;
    }
}
```

La lección es clara: Siempre usa try-with-resources en lugar de try-finally cuando trabajes con recursos que deben cerrarse. El código resultante es más corto y claro, y las excepciones que genera son más útiles. La declaración try-with-resources facilita la escritura de código correcto usando recursos que deben cerrarse, lo que era prácticamente imposible usando try-finally.

[CAPITULO_3](/capitulo_3.md)