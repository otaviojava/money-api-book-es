## Alguien pasó por esto antes? Conocimiento del API


En el capítulo anterior se había discutido las ventajas de traer el dinero como un tipo en vez de tratarlo solo como tipo primitivo. Pero será que nadie todavía paso por ese problema? El arte de “reinventar la rueda” además de ser muy costosa, ya que demanda mucho tiempo, pasa a ser muy peligrosa ya que se tiende a pasar por los mismos problemas que un programador con mas experiencia ya enfrentó. Una famosa frase de una conocida escritora brasilera *Clarice Lispector: *"Quien camina solo puede hasta llegar más rápido, pero aquel que va acompañado, con seguridad va ir más lejos."*, osea, la mejor estrategia es unirse a un framework que ya hace eso, así es posible intercambiar experiencias con programadores que ya pasaron por eso, no repetir los mismos errores, contribuir con esa herramienta, de esta forma, quedará cada vez mas robusta y con menos errores.

Por lo tanto, se puede unir a una solución preferida para desarrollar el tipo de dinero. Como encarar con este tipo de problema es cada vez más común en el desarrollo de software, que pasa si cada uno utiliza una solución específica? Por muchas razones se puede optar por una solución, por ejemplo, una buena documentación, licencia, razones comerciales, etc. Asi como integrar con cada una de ellas? En caso se escoja una es posible cambiar para otra? Sabemos que quedar enganchado en un vendedor no es algo bueno para los negocios. Con ese objetivo nace una especificación Java para afrontar el tipo dinero, la JSR 354. Esta tiene como principal objetivo tratar dinero. 

Una vez definiendo una interface común, cada empresa o desarrollador podrá optar por su solución favorita o cambiarla de forma transparente, para eso, basta que esa API implemente esa especificación. Ese comportamiento es bien semejante en un cambio de base de datos relacional en el mundo Java, en caso que la aplicación contenga una especificación y use interfaces de JDBC, se puede cambiar de base de datos tranquilamente, para eso, es necesario solo cambiar el driver, implementación, y todo continuará funcionando desde que tu aplicación use solo las interfaces.

Recordando el concepto de dinero, de forma resumida, dinero es compuesto por dos partes, la parte del valor que es la cantidad, asi representada de forma numérica, pero solo con ese valor no conseguimos hacer mucha cosa, necesitamos de la moneda. La moneda representa el “sistema de dinero” de uso común, especialmente dentro de una nación, siguiendo esa definición el real, nuevo sol, yen, peso, dólar y euros son tipos de monedas.

### Acceso al código fuente e instalación

Como toda especificación en Java, JSR, además del API esta posee una implementación de referencia, como el propio nombre dice, esa implementación servirá de base para las próximas implementaciones. En el caso de la JSR 354, money-api, una implementación de referencia es moneta (Para más informaciones en relación al código fuente del proyecto puedes acceder a [https://github.com/JavaMoney/jsr354-ri](https://github.com/JavaMoney/jsr354-ri)). Básicamente, es posible utilizar moneta de dos formas:

La primera de ellas es accesando a su repositorio de dependencia de maven ([http://mvnrepository.com/artifact/org.javamoney/moneta](http://mvnrepository.com/artifact/org.javamoney/moneta)), por lo que si tu proyecto usa maven, por ejemplo, basta adicionar una dependencia de moneta em su proyecto de la siguiente forma:

```
        <dependency>
            <groupId>org.javamoney</groupId>
            <artifactId>moneta</artifactId>
            <version>moneta_version</version>
        </dependency>
```

Para Gradle:
```
'org.javamoney:moneta:moneta_version'
```

Para Yvi:
```
<dependency org="org.javamoney" name="moneta" rev="moneta_version"/>
```


La otra forma es bajando el código-fuente y compilandolo, para eso será necesario seguir algunos pasos, mas básicamente es baja el parent o instalarlo y enseguida instalar moneta. Es importante recordar que para realizar ese procedimiento es necesario tener git, Java 8 y maven instalado y configurado en su máquina.


Bajar el código de javamoney-parent, para esto solo ejecute el siguiente comando:

```
git clone https://github.com/JavaMoney/javamoney-parent.git
```

A continuacion ingrese al directorio:
```
cd javamoney-parent
```

Compile el código-fuente e instalelo en su repositorio local, en nuestro caso también saltaremos las pruebas solo para que sea mas rápido:

```
mvn clean install -Dmaven.test.skip
```

Listo, una vez hecho todo eso el próximo paso será realizar la instalación de moneta.

Bajar el código de moneta, para esto solo ejecute el siguiente comando:

```
git clone git@github.com:JavaMoney/jsr354-ri.git
```

Ingrese al directorio:

```
cd jsr354-ri
```

Compile el código-fuente e instalelo en su repositorio local, en nuestro caso también saltaremos las pruebas solo para que sea mas rápido:

```
mvn clean install -Dmaven.test.skip
```


Listo, código-fuente instalado y podrá ser utilizado tranquilamente. En el caso del libro estamos utilizando como base un master del repositorio, osea, es recomendable que baje el código-fuente y lo instale localmente. Para tener acceso al código-fuente del ejemplo basta ingresar a:

* [https://github.com/otaviojava/money-api-book-samples](https://github.com/otaviojava/money-api-book-samples)


### Representando dinero y moneda con money-api



En money-api, será usado ese nombre en vez de JSR 354, también será necesario representar tanto el valor numérico en cuanto a la moneda. Para representar la moneda existe la interface ```CurrencyUnit``` esta necesita ser inmutable y thread-safe.

|Nombre del método| Descripción |Ejemplo|
| -- | -- | -- |
|```String getCurrencyCode()```|Retorna el código de la moneda, para las monedas que siguen el standard ISO serán retornado tres letras.|BRL para real brasileiro, USD para dólares americanos.
|```int getNumericCode()```|Retorna el código numérico de la moneda, asi como el código este posee tres dígitos.|986 para la moneda brasilera, 840 para dólares americanos.|
|``` int getDefaultFractionDigits()``` |Retorna el número de dígitos normalmente utilizado por la moneda.|BRL tiene dos y JPY no tiene.|


En este material la implementación utilizada será **moneta**, la implementación de referencia para esa especificación. Con este es posible crear una instancia de monedas de dos formas. La primera de ellas es utilizando el código de la moneda, si la moneda que siga el patrón de moneda será un ```String``` con tres letras.


```java
public class CurrencyExample1 {

    public static void main(String[] args) {

        CurrencyUnit currencyUnit = Monetary.getCurrency("BRL");
        String currencyCode = currencyUnit.getCurrencyCode();//BRL
        int numericCurrencyCode = currencyUnit.getNumericCode();//986
        int fractionDigits = currencyUnit.getDefaultFractionDigits();//2

    }
}
```

Otra forma simple es utilizando la clase ```Locale```, esa creación es muy interesante, por ejemplo, en una aplicación web de compras en que a partir de ```Locale``` de la solicitud será posible saber cual es la moneda del usuario que está accediendo a la aplicación.

```java

public class CurrencyExample2 {

    public static void main(String[] args) {
        CurrencyUnit currencyUnit = Monetary.getCurrency(Locale.US);
        String currencyCode = currencyUnit.getCurrencyCode();//USD
        int numericCurrencyCode = currencyUnit.getNumericCode();//840
        int fractionDigits = currencyUnit.getDefaultFractionDigits();//2
    }
}

```

Definido la representación de moneda el próximo paso será la representación del valor monetario, para eso, existe la interface ```MonetaryAmount```, uma característica importante es que todas las implementaciones necesitan ser inmutables y thread-safe. 

|Método| Descripción|
| -- | -- |
|```<R> R query(MonetaryQuery<R> query)```|Realiza query con valor monetário.|
|```MonetaryAmount with(MonetaryOperator operator)```|Realiza operaciones con el importe monetário.|
|```boolean isGreaterThan(MonetaryAmount amount)```|Retorna verdadero si la instancia es mayor que el valor pasado en el  parametro, asi sean iguales este retorna falso.|
|```boolean   isGreaterThanOrEqualTo(MonetaryAmount amount)```|Retorna verdadero si la instancia es mayor o igual que el valor pasado en el parametro, asi sean iguales este retorna verdadero.|
|```boolean isLessThan(MonetaryAmount amount)```|Retorna verdadero si una instancia es menor que el valor pasado en el  parametro, asi sean iguales este retorna falso.|
|```isLessThanOrEqualTo(MonetaryAmount amt)```|Retorna verdadero si una instancia es menor o igual que el valor pasado en el parametro, asi sean iguales este retorna verdadero.|
|```boolean isEqualTo(MonetaryAmount amount)```|Retorna verdadero si una instancia es igual al valor monetário pasado en el parametro.|
|```boolean isNegative()```|Retorna verdadero si es negativo|
|```boolean isNegativeOrZero()```|Retorna verdadero si es negativo o cero|
|```isPositive()```|Retorna verdadero si es positivo|
|```boolean isPositiveOrZero()```|Retorna verdadero si es positivo o igual a cero|
|```isZero()```|Verifica si el valor de dinero es cero.|
|```MonetaryAmount add(MonetaryAmount amount)```|Realiza la suma y retorna el resultado|
|```MonetaryAmount subtract(MonetaryAmount amount)```|Realiza la subtracción y retorna el resultado|
|```MonetaryAmount multiply(Number multiplicand)```|Realiza la multiplicación y retorna el resultado|
|```MonetaryAmount divide(Number divisor)```|Realiza a división|
|```MonetaryAmount remainder(Number divisor)```|Realiza la división y retorna el resto|
|```MonetaryAmount negate()```|Realiza la negación del importe monetario, osea, -este.
|```getCurrency()```|Retorna dinero de moneda.|

Dentro de moneta existen tres implementaciones para la interface ```MonetaryAmount```:


1. 
Money: la implementación estándar, ésta representa el valor numérico con BigDecimal.
1. 
RoundedMoney: así como la implementación Money, representa el valor numérico con BigDecimal, la diferencia entre ellos es que con RoundedMoney es posible recibir un MonetaryOperator para ser llamada a cada operación, por ejemplo, a cada operación aritmética realizar una operación de redondeo.

1. 
FastMoney: la implementación que representa el valor número con primitivo long, de las implementaciones presentadas ésta es la más rápida, cerca de quince veces más rápidas que las otras dos, además de ser más ligero en la creación. Sin embargo, esta posee una mayor limitación en relación a la precisión, si es necesario trabajar con esa precisión, las operaciones no puede exceder de 5 puntos decimales.

### Métodos de creación


Para crear una instancia de ```MonetaryAmount```, todas las implementaciones siguen el mismo standard de nomenclatura utilizando, con una pequeña excepción en ```RoundedMoney``` una vez que el pueda recibir un ```MonetaryOperator``` para trabajar como “redondeador” en cada operación. Listando los mas importantes tenemos tres métodos, que son:

* El método **of** pasando un number y un código de moneda.
* El método **zero** pasando un CurrencyUnit.
* El método **ofMinor** pasando un long y una moneda, ese long será tratado como centavos y será convertido llevando en consideración la moneda, por ejemplo, 200 cents equivalen a dos dólares.


```java
public class MethodsCreationMoney {

    public static void main(String[] args) {
        CurrencyUnit currency = Monetary.getCurrency("BRL");
        MonetaryAmount money = Money.of(BigDecimal.TEN, currency); //BRL 10
        MonetaryAmount zero = Money.zero(currency);//BRL 0
        MonetaryAmount moneyFromCurrencyCode = Money.of(10, "USD");//USD 10
        MonetaryAmount moneyFromCents = Money.ofMinor(currency, 100_00);//BRL 10
    }
}
```


```java
public class MethodsCreationsFastMoney {

    public static void main(String[] args) {
        CurrencyUnit currency = Monetary.getCurrency("BRL");
        MonetaryAmount money = FastMoney.of(BigDecimal.TEN, currency); //BRL 10
        MonetaryAmount zero = FastMoney.zero(currency);//BRL 0
        MonetaryAmount moneyFromCurrencyCode = FastMoney.of(10, "USD");//USD 10
        MonetaryAmount moneyFromCents = FastMoney.ofMinor(currency, 100_00);//BRL 10
    }
}
```


```java
public class MethodsCreationRoundedMoney {

    public static void main(String[] args) {
        CurrencyUnit currency = Monetary.getCurrency("BRL");
        MonetaryAmount money = RoundedMoney.of(BigDecimal.TEN, currency); //BRL 10
        MonetaryAmount zero = RoundedMoney.zero(currency);//BRL 0
        MonetaryAmount moneyFromCurrencyCode = RoundedMoney.of(10, "USD");//USD 10
        MonetaryAmount moneyFromCents = RoundedMoney.ofMinor(currency, 100_00);//BRL 10
    }
}
```
 
#### Métodos de creación para RoundedMoney



Además de los métodos comunes de construcción la clase ```RoundedMoney```, posee otras formas de parámetros para que sea posible informar a ```MonetaryOperator``` para ser ejecutado después de cada operación de ```MonetaryAmount```, vale recordar, la principal característica de esa clase es realizar ese tipo de operación, asi no sea necesario, otra implementación es recomendada.


```java
public class RoundedMoneyCreation2 {

    public static void main(String[] args) {
        CurrencyUnit currency = Monetary.getCurrency("BRL");
        MonetaryAmount money = RoundedMoney.of(BigDecimal.TEN, currency, MonetaryOperators.rounding()); //BRL 10
        MonetaryAmount zero = RoundedMoney.zero(currency);//BRL 0
        MonetaryAmount moneyFromCurrencyCode = RoundedMoney.of(10, "USD");//USD 10
        MonetaryAmount moneyFromCents = RoundedMoney.ofMinor(currency, 100_00);//BRL 10
    }
}
```

### Operaciones Aritméticas


Con ```MonetaryAmount``` es posible realizar operaciones como substracción y suma, subrayando que las implementaciones son inmutables, osea, el resultado resultará en una nueva instancia. Al realizar operaciones que reciben un ```MonetaryAmount``` el resultado será también un ```MonetaryAmount```, pero de la implementación de la instancia que llamó al metodo.

```java
public class ArithmeticOperations {

    public static void main(String[] args) {
        CurrencyUnit currency = Monetary.getCurrency("BRL");
        MonetaryAmount money = Money.of(BigDecimal.TEN, currency);
        MonetaryAmount money2 = FastMoney.of(BigDecimal.TEN, currency);
        MonetaryAmount addResult = money.add(money2);//BRL 20 Money implementation
        MonetaryAmount subtractResult = money2.subtract(addResult);//BRL -10 FastMoney implementation
    }
}
```

Para las operaciones de multiplicación, división y resto es necesário pasar un parametro del tipo ```Number```.

```java
public class ArithmeticOperations2 {

    public static void main(String[] args) {
        CurrencyUnit currency = Monetary.getCurrency("BRL");
        MonetaryAmount money = Money.of(100, currency);
        Number number = 20;
        MonetaryAmount divideResult = money.divide(number);//BRL 5
        MonetaryAmount remainderResult = money.remainder(30);//BRL 10
        MonetaryAmount resultMultiply = money.multiply(5);//BRL 500
    }
}
```

También es posible realizar operaciones de negación con ```MonetaryAmount```.



```java
public class ArithmeticOperations3 {

    public static void main(String[] args) {
        CurrencyUnit currency = Monetary.getCurrency("BRL");
        MonetaryAmount money = Money.of(100, currency);
        MonetaryAmount negateResult = money.negate();//BRL -100
        MonetaryAmount plusResult = money.plus();//BRL 100
    }
}
```

### Operaciones booleanas

Con MonetaryAmount es posible realizar comparaciones en relaciones a otros ```MonetaryAmount```, con este ésto posible, por ejemplo, verificar si un dinero es mayor, menor o igual a otro.


```java
    public static void main(String[] args) {
        CurrencyUnit currency = Monetary.getCurrency("BRL");
        MonetaryAmount money = Money.of(10, currency);
        MonetaryAmount money2 = Money.of(20, currency);
        boolean equalsToResult = money.isEqualTo(money2);//false
        boolean greaterThan = money.isGreaterThan(money2);//false
        boolean greaterThanOrEqualTo = money.isGreaterThanOrEqualTo(money2);//false
        boolean lessThan = money.isLessThan(money2);//true
        boolean lessThanOrEqualTo = money.isLessThanOrEqualTo(money2);//true
        boolean negative = money.isNegative();//false
        boolean negativeOrZero = money.isNegativeOrZero();//false
        boolean positive = money.isPositive();//true
        boolean positiveOrZero = money.isPositiveOrZero();//true
        boolean zero = money.isZero();//false
    }
}
```


### NumberValue: Una representación de la parte numérica de money


A veces es necesario recuperar y manipular la información de dinero de forma separada, para eso, una interface dispone de un método para recuperar tanto la moneda como el valor monetário. Para la moneda este obviamente devuelve la interface ```CurrencyUnit``` y para representar el valor numérico es devuelto la clase ```NumberValue```. Esta clase es hija de ```Number``` de Java, asi es posible recuperar para los tipos primitivos básicos de Java (```int```, ```long```, ```float```, ```double```, etc.).


```java
public class RetrieveInformationMethods {

    public  static  void main(String[] args) {

        MonetaryAmount money = Money.of(10, Monetary.getCurrency("BRL"));
        CurrencyUnit currency = money.getCurrency();
        Number number = money.getNumber();

    }
}
```

Además de eso, con ```NumberValue``` es posible realizar algunas operaciones más, por ejemplo, es posible definir cuál clase será recuperada a partir de ```NumberValue```, desde que esa clase sea hija de la clase ```Number```. En el caso de la implementación de referencia, para esto serán todas las clases que son ```Number``` y estan dentro de JDK.


```java
public class RetrieveInformationMethods2 {

    public  static  void main(String[] args) {


        MonetaryAmount money = Money.of(BigDecimal.valueOf(10.21), Monetary.getCurrency("BRL"));
        NumberValue number = money.getNumber();
        int precision = number.getPrecision();//4
        int scale = number.getScale();//2
        long amountFractionDenominator = number.getAmountFractionDenominator();//21
        long amountFractionNumerator = number.getAmountFractionNumerator();//10
        Class<?> numberType = number.getNumberType();//java.math.BigDecimal
        BigDecimal value = number.numberValue(BigDecimal.class);
        Integer integerValue = number.numberValue(Integer.class);


    }
}
```




