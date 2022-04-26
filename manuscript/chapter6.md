## Trabajando con Streams

Una vez que la API nació despues de Java 8, es natural que este tenga soporte a algunos recursos de Java 8, como ya vimos en el capítulo anterior, la búsqueda de una cotización es dada a partir de un ```LocalDate```, sin mencionar en el soporte la nueva estructura de datos mejoró la experiencia en trabajar con listas, el **Stream**. Trabajar con colecciones es muy importante y común, por ejemplo, una lista de produtos que serán cobrados es natural que sea impreso el valor total de esa compra. Para trabajar con Stream la implementación de referencia tiene la clase ```MonetaryFunctions```.

### Ordenando una lista monetaria:


Dentro de la clase `MonetaryFunctions` es posible ordenar por la moneda, por el valor numérico solo, además del valor de dinero, teniendo en consideración la cotización de la moneda, de forma ascendente y descendente. 
 
 #### Realizando ordenación con moneda

En caso de la ordenación de la moneda es teniendo en consideración el código de la moneda. Por ejemplo, una lista con las monedas `USD`, `EUR`, `BRL` retornará `BRL`, `EUR` y `USD` de forma ascendente y USD, EUR, BRL de forma descendente.

```java
public class SortMonetaryAmountCurrency {

    public static void main(String[] args) {
        CurrencyUnit dollar = Monetary.getCurrency("USD");
        CurrencyUnit euro = Monetary.getCurrency("EUR");
        CurrencyUnit real = Monetary.getCurrency("BRL");

        MonetaryAmount money = Money.of(9, euro);
        MonetaryAmount money2 = Money.of(10, dollar);
        MonetaryAmount money3 = Money.of(11, real);

        List<MonetaryAmount> resultAsc = Stream.of(money, money2, money3)
                .sorted(MonetaryFunctions
                        .sortCurrencyUnit()).collect(Collectors.toList());//[BRL 11, EUR 9, USD 10]
        List<MonetaryAmount> resultDesc = Stream.of(money, money2, money3)
                .sorted(MonetaryFunctions
                        .sortCurrencyUnitDesc()).collect(Collectors.toList());//[USD 10, EUR 9, BRL 11]

    }
}
```

#### Realizando ordenación con valor numérico

La ordenación por el valor numérico ignora la moneda y ordena solo teniendo en consideración el valor monetário, cabe resaltar, que esa ordenación no realiza cotización de valores, en otras palabras, el valor de diez reales tendrá el mismo valor que diez dolares. También es posible retornar de forma ascendente y descendente.

```java
public class SortMonetaryAmountNumber {

    public static void main(String[] args) {
        CurrencyUnit dollar = Monetary.getCurrency("USD");
        CurrencyUnit euro = Monetary.getCurrency("EUR");
        CurrencyUnit real = Monetary.getCurrency("BRL");

        MonetaryAmount money = Money.of(9, euro);
        MonetaryAmount money2 = Money.of(10, dollar);
        MonetaryAmount money3 = Money.of(11, real);

        List<MonetaryAmount> resultAsc = Stream.of(money, money2, money3)
                .sorted(MonetaryFunctions
                        .sortNumber()).collect(Collectors.toList());//[EUR 9, USD 10, BRL 11]
        List<MonetaryAmount> resultDesc = Stream.of(money, money2, money3)
                .sorted(MonetaryFunctions
                        .sortNumberDesc()).collect(Collectors.toList());//[BRL 11, USD 10, EUR 9]
    }
}
```
#### Realizando ordenación teniendo en consideración la cotización


También es posible realizar una ordenación de forma creciente y decreciente teniendo en consideración la cotización de la moneda. Para esto basta pasar una implementación de `ExchangeRateProvider`. Por ejemplo, dado una lista con diez dolares, once reales y nueve euros, retornará de forma ascendente el valor de once reales, diez dólares y nueve euros teniendo en consideración que por la cotización el dólar es más valioso que el real y menos valioso que el euro.


```java
public class SortMonetaryAmountExchange {

    public static void main(String[] args) {
        CurrencyUnit dollar = Monetary.getCurrency("USD");
        CurrencyUnit euro = Monetary.getCurrency("EUR");
        CurrencyUnit real = Monetary.getCurrency("BRL");

        MonetaryAmount money = Money.of(9, euro);
        MonetaryAmount money2 = Money.of(10, dollar);
        MonetaryAmount money3 = Money.of(11, real);

        ExchangeRateProvider provider =
                MonetaryConversions.getExchangeRateProvider(ExchangeRateType.IMF);

    
        List<MonetaryAmount> resultAsc = Stream.of(money, money2, money3)
                .sorted(MonetaryFunctions
                        .sortValiable(provider))
                .collect(Collectors.toList());//[BRL 11, EUR 9, USD 10]

        List<MonetaryAmount> resultDesc = Stream.of(money, money2, money3)
                .sorted(MonetaryFunctions
                        .sortValiableDesc(provider)).collect(Collectors.toList());//[USD 10, EUR 9, BRL 11]

    }
}
```

#### Uniendo las ordenaciones



Solo como recordatorio, ya que este recurso no es de money-api y si de Java 8, es posible mezclar mas de un ordenador, para eso basta utilizar el método **thenComparing**. Básicamente el hace la ordenación y si los valores tengan el mismo peso, al usar el compare devuelva el valor zero, este usará el otro ordenador, asi el orden que fuera definido o sort influenciará en el resultado de la ordenación.


```java
public class SortMixMonetaryAmountNumberCurrency {

    public static void main(String[] args) {
        CurrencyUnit dollar = Monetary.getCurrency("USD");
        CurrencyUnit euro = Monetary.getCurrency("EUR");
        CurrencyUnit real = Monetary.getCurrency("BRL");

        MonetaryAmount money = Money.of(10, euro);
        MonetaryAmount money2 = Money.of(10, dollar);
        MonetaryAmount money3 = Money.of(10, real);
        MonetaryAmount money4 = Money.of(9, real);
        MonetaryAmount money5 = Money.of(8, dollar);

        List<MonetaryAmount> resultAsc = Stream.of(money, money2, money3, money4, money5)
                .sorted(MonetaryFunctions
                        .sortNumber().thenComparing(MonetaryFunctions.sortCurrencyUnit()))
                .collect(Collectors.toList());//[USD 8, BRL 9, BRL 10, EUR 10, USD 10]
        List<MonetaryAmount> resultDesc = Stream.of(money, money2, money3, money4, money5)
                .sorted(MonetaryFunctions
                        .sortNumberDesc().thenComparing(MonetaryFunctions.sortCurrencyUnitDesc()))
                .collect(Collectors.toList());//[USD 10, EUR 10, BRL 10, BRL 9, USD 8]
        //using currency first
        List<MonetaryAmount> resultCurrencyAsc = Stream.of(money, money2, money3, money4, money5)
                .sorted(MonetaryFunctions
                        .sortCurrencyUnit().thenComparing(MonetaryFunctions.sortNumber()))
                .collect(Collectors.toList());//[BRL 9, BRL 10, EUR 10, USD 8, USD 10]
        List<MonetaryAmount> resultCurrencyDesc = Stream.of(money, money2, money3, money4, money5)
                .sorted(MonetaryFunctions
                        .sortCurrencyUnitDesc().thenComparing(MonetaryFunctions.sortNumberDesc()))
                .collect(Collectors.toList());//[USD 10, USD 8, EUR 10, BRL 10, BRL 9]

    }
}
```

### Métodos de reducción


La reducción es el proceso en que, dado una lista de valores este deberá retornar una o ninguna salida, básicamente, a partir de una lista lo transforma para solo un elemento, o la ausencia de este.

#### Sumatoria

La suma o la reducción por la suma es definido en: Dado una lista de `MonetaryAmount` este retornará un elemento con la sumatoria de esos valores.


```java
public class ReduceSumMonetaryAmount {

    public static void main(String[] args) {
        CurrencyUnit dollar = Monetary.getCurrency("USD");

        MonetaryAmount money = Money.of(10, dollar);
        MonetaryAmount money2 = Money.of(10, dollar);
        MonetaryAmount money3 = Money.of(10, dollar);
        MonetaryAmount money4 = Money.of(9, dollar);
        MonetaryAmount money5 = Money.of(8, dollar);

        Optional<MonetaryAmount> result = Stream.of(money, money2, money3, money4, money5).reduce(MonetaryFunctions.sum());
        result.ifPresent(System.out::println);//USD 47

    }
}
```

Resaltando algo, en caso tenga una moneda diferente en la sumatoria este retornará una excepción.

```java
public class ReduceSumMonetaryAmountError {

    public static void main(String[] args) {
        CurrencyUnit dollar = Monetary.getCurrency("USD");
        CurrencyUnit real = Monetary.getCurrency("BRL");

        MonetaryAmount money = Money.of(10, dollar);
        MonetaryAmount money2 = Money.of(10, dollar);
        MonetaryAmount money3 = Money.of(10, real);
        MonetaryAmount money4 = Money.of(9, dollar);
        MonetaryAmount money5 = Money.of(8, dollar);

        Optional<MonetaryAmount> result = Stream.of(money, money2, money3, money4, money5).reduce(
                MonetaryFunctions.sum());//javax.money.MonetaryException: Currency mismatch: BRL/USD
        
    }
}
```

En caso sea necesario sumar y convertir para una específica moneda, Moneta da soporte para eso, basta proporcionar una implementación de `ExchangeRateProvider` y también la moneda en la que todos los valores serán convertidos.

```java
public class ReduceSumMonetaryAmountExchange {

    public static void main(String[] args) {
        CurrencyUnit dollar = Monetary.getCurrency("USD");
        CurrencyUnit real = Monetary.getCurrency("BRL");
        ExchangeRateProvider provider = MonetaryConversions.getExchangeRateProvider(ExchangeRateType.IMF);

        MonetaryAmount money = Money.of(10, dollar);
        MonetaryAmount money2 = Money.of(10, dollar);
        MonetaryAmount money3 = Money.of(10, real);
        MonetaryAmount money4 = Money.of(9, dollar);
        MonetaryAmount money5 = Money.of(8, dollar);

        Optional<MonetaryAmount> result = Stream.of(money, money2, money3, money4, money5).reduce(
                MonetaryFunctions.sum(provider, dollar));//javax.money.MonetaryException: Currency mismatch: BRL/USD
        result.ifPresent(System.out::println);//money converted in dollar
        
    }
}
```

#### Valor mínimo y valor máximo

Es posible realizar la reducción por el valor máximo y mínimo, por ejemplo, dado una lista es posible retornar el mayor o el menor valor de la lista.


```java
public class ReduceMaxMinMonetaryAmount {

    public static void main(String[] args) {
        CurrencyUnit dollar = Monetary.getCurrency("USD");


        MonetaryAmount money = Money.of(10, dollar);
        MonetaryAmount money2 = Money.of(10, dollar);
        MonetaryAmount money3 = Money.of(10, dollar);
        MonetaryAmount money4 = Money.of(9, dollar);
        MonetaryAmount money5 = Money.of(8, dollar);

        Optional<MonetaryAmount> max = Stream.of(money, money2, money3, money4, money5).reduce(
                MonetaryFunctions.max());//javax.money.MonetaryException: Currency mismatch: BRL/USD
        max.ifPresent(System.out::println);//USD 10

        Optional<MonetaryAmount> min = Stream.of(money, money2, money3, money4, money5).reduce(
                MonetaryFunctions.min());
        min.ifPresent(System.out::println);//USD 8
        
    }
}
```

Si es realizado la operación de mínimo y máximo con monedas diferentes, sucederá una excepción. Es posible pasar un `ExchangeRateProvider` para realizar la conversión y entonces la comparación.


```java
public class ReduceMaxMinMonetaryAmountExchange {

    public static void main(String[] args) {
        CurrencyUnit dollar = Monetary.getCurrency("USD");
        CurrencyUnit euro = Monetary.getCurrency("EUR");
        ExchangeRateProvider provider = MonetaryConversions.getExchangeRateProvider(ExchangeRateType.IMF);

        MonetaryAmount money = Money.of(10, dollar);
        MonetaryAmount money2 = Money.of(10, euro);
        MonetaryAmount money3 = Money.of(10, dollar);
        MonetaryAmount money4 = Money.of(9, euro);
        MonetaryAmount money5 = Money.of(8, dollar);

        Optional<MonetaryAmount> max = Stream.of(money, money2, money3, money4, money5).reduce(
                MonetaryFunctions.max(provider));//javax.money.MonetaryException: Currency mismatch: BRL/USD
        max.ifPresent(System.out::println);//EUR 10

        Optional<MonetaryAmount> min = Stream.of(money, money2, money3, money4, money5).reduce(
                MonetaryFunctions.min(provider));
        min.ifPresent(System.out::println);//USD 8
        
    }
}
```

### Predicates


Operación de predicate es cuando una entrada o retorno es un valor booleano, osea, *verdadero* o *falso*. Dentro del Stream el predicate puede ser utilizado como filtro, filtrar por una moneda específica, o para match, se verifica si existe algun elemento de la lista o todos pertenecen a esa condición.


```java
public class BooleanMonetaryAmount {

    public static void main(String[] args) {
        CurrencyUnit dollar = Monetary.getCurrency("USD");
        CurrencyUnit real = Monetary.getCurrency("BRL");


        MonetaryAmount money = Money.of(10, dollar);
        MonetaryAmount money2 = Money.of(10, real);
        MonetaryAmount money3 = Money.of(10, dollar);
        MonetaryAmount money4 = Money.of(9, real);
        MonetaryAmount money5 = Money.of(8, dollar);

        Stream<MonetaryAmount> justDollars = Stream.of(money, money2, money3, money4, money5)
                .filter(MonetaryFunctions.isCurrency(dollar));

        boolean anyMatch = Stream.of(money, money2, money3, money4, money5)
                .anyMatch(MonetaryFunctions.isCurrency(dollar));//true

        boolean allMatch = Stream.of(money, money2, money3, money4, money5)
                .allMatch(MonetaryFunctions.isCurrency(dollar));//true
    }
}
```

#### Predicate con monedas


Con Moneta es posible realizar predicate a partir de la moneda, usando inclusive y exclusive. Ambos utilizan varargs, osea, es posible adicionar n elementos en la condición:


* `isCurrency(CurrencyUnit... currencies)`: Retorna verdadero si MonetaryAmount tenga una de las monedas especificadas.	
* `filterByExcludingCurrency(CurrencyUnit... currencies)`: Retorna verdadero si MonetaryAmount no tenga ninguna de las monedas especificadas.


```java
public class PredicateMonetaryAmountCurrency {

    public static void main(String[] args) {
        CurrencyUnit dollar = Monetary.getCurrency("USD");
        CurrencyUnit real = Monetary.getCurrency("BRL");
        CurrencyUnit euro = Monetary.getCurrency("EUR");


        MonetaryAmount money = Money.of(10, dollar);
        MonetaryAmount money2 = Money.of(10, real);
        MonetaryAmount money3 = Money.of(10, dollar);
        MonetaryAmount money4 = Money.of(9, real);
        MonetaryAmount money5 = Money.of(8, dollar);

        List<MonetaryAmount> justDollar = Stream.of(money, money2, money3, money4, money5)
                .filter(MonetaryFunctions.isCurrency(dollar)).collect(Collectors.toList());//[USD 10, USD 10, USD 8]

        boolean anyDollar = Stream.of(money, money2, money3, money4, money5)
                .anyMatch(MonetaryFunctions.isCurrency(dollar));//true

        boolean allDollar = Stream.of(money, money2, money3, money4, money5)
                .allMatch(MonetaryFunctions.isCurrency(dollar));//false

        List<MonetaryAmount> notDollar = Stream.of(money, money2, money3, money4, money5)
                .filter(MonetaryFunctions.filterByExcludingCurrency(dollar)).collect(Collectors.toList());//[BRL 10, BRL 9]

        boolean anyMatch = Stream.of(money, money2, money3, money4, money5)
                .anyMatch(MonetaryFunctions.filterByExcludingCurrency(euro));//true

        boolean allMatch = Stream.of(money, money2, money3, money4, money5)
                .allMatch(MonetaryFunctions.filterByExcludingCurrency(euro));//true

       
    }
}
```

#### Predicate con valor numérico


Además de operaciones con moneda es posible realizar condiciones con valor numérico de `MonetaryAmount`, los métodos tienen el mismo comportamento de la interface `MonetaryAmount`:


* `MonetaryFunctions.isLessThanOrEqualTo(monetaryAmount)`
* `MonetaryFunctions.isLessThan(monetaryAmount)`
* `MonetaryFunctions.isGreaterThan(monetaryAmount)`
* `MonetaryFunctions.isGreaterThanOrEqualTo(monetaryAmount)`
* `isBetween(MonetaryAmount min, MonetaryAmount max)`


```java
public class PredicateMonetaryAmountNumberValue {

    public static void main(String[] args) {
        CurrencyUnit dollar = Monetary.getCurrency("USD");



        MonetaryAmount money = Money.of(10, dollar);
        MonetaryAmount money2 = Money.of(10, dollar);
        MonetaryAmount money3 = Money.of(10, dollar);
        MonetaryAmount money4 = Money.of(9, dollar);
        MonetaryAmount money5 = Money.of(8, dollar);

        List<MonetaryAmount> greaterThanZero = Stream.of(money, money2, money3, money4, money5)
                .filter(MonetaryFunctions.isGreaterThan(Money.zero(dollar)))
                .collect(Collectors.toList());//[USD 10, USD 10, USD 10, USD 9, USD 8]

        boolean hasAnyGreaterThanZero = Stream.of(money, money2, money3, money4, money5)
                .anyMatch(MonetaryFunctions.isGreaterThan(Money.zero(dollar)));//true

        boolean allBetweenZeroAndTen = Stream.of(money, money2, money3, money4, money5)
                .allMatch(MonetaryFunctions.isBetween(Money.zero(dollar),
                        Money.of(BigDecimal.TEN, dollar)));//true

    System.out.println(greaterThanZero);
    }
}
```

#### Interactuando con Predicates


Asi como sort, también es posible realizar interacción con predicate, utilizando operaciones booleanas, asi tenemos los  métodos, **negate**, **and**, **or**.

```java
public class PredicateMonetaryAmountMix {

    public static void main(String[] args) {
        CurrencyUnit dollar = Monetary.getCurrency("USD");
        CurrencyUnit real = Monetary.getCurrency("BRL");


        MonetaryAmount money = Money.of(10, dollar);
        MonetaryAmount money2 = Money.of(10, dollar);
        MonetaryAmount money3 = Money.of(10, dollar);
        MonetaryAmount money4 = Money.of(9, dollar);
        MonetaryAmount money5 = Money.of(8, dollar);
        MonetaryAmount money6 = Money.of(8, dollar);

		List<MonetaryAmount> greaterThanZeroAndIsReal = Stream
				.of(money, money2, money3, money4, money5, money6)
				.filter(MonetaryFunctions.isGreaterThan(Money.zero(dollar))
						.and(MonetaryFunctions.isCurrency(real)))
				.collect(Collectors.toList());//[]
		List<MonetaryAmount> greaterThanZeroOrIsReal = Stream
				.of(money, money2, money3, money4, money5, money6)
				.filter(MonetaryFunctions.isGreaterThan(Money.zero(dollar))
						.or(MonetaryFunctions.isCurrency(real)))
				.collect(Collectors.toList());//[USD 10, USD 10, USD 10, USD 9, USD 8, USD 8]

		
		List<MonetaryAmount> notGreaterThan = Stream
				.of(money, money2, money3, money4, money5, money6)
				.distinct()
				.filter(MonetaryFunctions.isGreaterThan(Money.of(9, dollar))
						.negate())
				.collect(Collectors.toList());//[USD 9, USD 8]
    }
}
```
