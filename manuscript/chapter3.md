## Manipulando y seleccionando información de MonetaryAmount


Además de realizar operaciones básicas como las operaciones aritméticas, algunas veces es necesario seleccionar y convertir informaciones de dinero, por ejemplo, seleccionar la menor parte de centavos, obtener solo los centavos, o solo la mayor parte de dinero. Con ese objetivo existe las interfaces ```MonetaryOperator``` y ```MonetaryQuery```. Ambas interfaces poseen solo un método para ser implementados, osea, son interfaces funcionales.


```java

@FunctionalInterface
public interface MonetaryOperator{

    MonetaryAmount apply(MonetaryAmount amount);
}


@FunctionalInterface
public interface MonetaryQuery<R>{

    R queryFrom(MonetaryAmount amount);
}

```

### MonetaryOperator


```MonetaryOperator``` es una interface funcional que recibe un ```MonetaryAmount``` y retorna un ```MonetaryAmount```. Esa interface es muy discutida al hablarse sobre la implementación de ```RoundedMoney``` que la utiliza como agente redondeador. Con esta interface es posible realizar operaciones de redondeo, retornar parte de dinero, el doble del valor, etc. 

```java
public class MonetaryOperatorExamples {

    public  static void main(String[] args) {
        MonetaryOperator doubleOperator = m -> m.multiply(2);
        MonetaryOperator halfOperator = m -> m.divide(2);
        MonetaryOperator operator = m -> {
            if(m.isPositive()){
                return m.multiply(2);
            }
            return m.divide(2);
        };
    }
}
```

Para ejecutarlo basta llamar el método apply de la interface o llamar el método **with** dentro de ```MonetaryAmount```.


```java
public class HelloMonetaryOperator {

    public  static void main(String[] args) {
        CurrencyUnit currency = Monetary.getCurrency("BRL");
        MonetaryAmount money = FastMoney.of(10, currency);
        MonetaryOperator doubleOperator = m -> m.multiply(2);
        MonetaryAmount result = doubleOperator.apply(money);//BRL 20.00000
        MonetaryAmount result2 = result.with(doubleOperator);//BRL 40.00000
    }
}
```


Moneta trae por estándar algunas implementaciones de `MonetaryOperator`, la clase `MonetaryOperators`. esta es una clase utilitaria que trae algunas funcionalidades importantes y algunas veces muy cotidiano dentro de la vida de un desarrollador Java, como:



* **reciprocal()** Retorna dinero como reciprocal, multiplicado por el valor inverso (1/n).
* **permil(Number number)** retorna el valor permil de dinero, por ejemplo, **permil(10)** de `EUR 2.35` retornará EUR `0.0235`.
* **percent(Number number)** retorna el percentual de dinero, por ejemplo, **percent(10)** de `EUR 200.00` retornará `EUR 20.00`.
* **minorPart()** retorna el valor que se encuentra a la derecha del punto decimal, por exemplo, la menor parte de `EUR 2.35` es ```EUR 0.35```.
* **majorPart()** retorna el valor entero de dinero, por ejemplo, la menor parte de `EUR 2.35` es `EUR 2`.
* **rounding()** realiza el proceso de redondeo de dinero, para saber el número de decimales despues del punto es recuperado la información del método getDefaultFractionDigits() de la interface CurrencyUnit.
* **exchange(CurrencyUnit currency)** dado un Dinero ese operador realizará el intercambio de la moneda, osea, este solo va intercambiar la moneda no llevando en consideración su cotización, por ejemplo, `EUR 2.35` **exchange('BRL')** retornará `BRL 2.35`. Este método está en la clase `ConversionOperators`.

```java

public class MonetaryOperatorsExample {

    public static void main(String[] args) {
        CurrencyUnit currency = Monetary.getCurrency("BRL");
        CurrencyUnit dollar = Monetary.getCurrency(Locale.US);
        
        MonetaryAmount money = Money.of(120.231, currency);
        
        MonetaryAmount majorPartResult = money.with(MonetaryOperators.majorPart());//BRL 120
        MonetaryAmount minorPartResult = money.with(MonetaryOperators.minorPart());//BRL 0.231
        MonetaryAmount percentResult = money.with(MonetaryOperators.percent(20));//BRL 24.0462
        MonetaryAmount permilResult = money.with(MonetaryOperators.permil(100));//BRL 12.0231
        MonetaryAmount roundingResult = money.with(MonetaryOperators.rounding());//BRL 120.23
        MonetaryAmount resultExchange = money.with(ConversionOperators.exchange(dollar));//USD 120.231
    }
}
```

### MonetaryQuery


```MonetaryQuery``` semejante a ```MonetaryOperator``` es una interface funcional que recibe un ```MonetaryOperator```, la diferencia está en el retorno, este puede devolver cualquier tipo de objeto a partir de *generics*. Con ```MonateryQuery``` es posible recuperar algunas informaciones en el ```MonetaryAmount```, por ejemplo, el código de la moneda, solo el número en el  formato long o en BigDecimal.

```java
public class MonetaryQueryExamples {

    public static void main(String[] args) {
        MonetaryQuery<Long> longQuery = m -> m.getNumber().longValue();
        MonetaryQuery<String> currencyCodeQuery = m -> m.getCurrency().getCurrencyCode();
        MonetaryQuery<Integer> fractionDigits = m -> m.getCurrency().getDefaultFractionDigits();
        

    }
}
```

Para ejecutarlo basta llamar al método apply de la interface o llamar al método “with” dentro de MonetaryAmount.


```java
public class HelloMonetaryQuery {

    public  static void main(String[] args) {
        CurrencyUnit currency = Monetary.getCurrency("BRL");
        MonetaryAmount money = FastMoney.of(10, currency);
        MonetaryQuery<String> currencyCodeQuery = m -> m.getCurrency().getCurrencyCode();
        String result = currencyCodeQuery.queryFrom(money);//BRL
        String result2 = money.query(currencyCodeQuery);//BRL
    }
}
```


Moneta trae como estándar algunas implementaciones de ```MonetaryQuery```, la clase ```MonetaryQueries```. Esta es una clase utilitaria que trae algunas funcionalidades importantes y algunas veces muy cotidianas dentro de la vida de un desarrollador Java, como:

* **MonetaryQuery<Long> extractMajorPart()** recupera la mayor parte de dinero, por ejemplo, `EUR 2.35` retornará `2`.
* **MonetaryQuery<Long> convertMinorPart()** recupera el valor monetario, o convirtiendo para la menor parte, centavos de dinero, por ejemplo, `USD 2.35` será retornado `235`.


```java
public class MonetaryQueriesExample {

    public static void main(String[] args) {
        CurrencyUnit currency = Monetary.getCurrency(Locale.US);

        MonetaryAmount money = Money.of(120.23, currency);
        Long moneyInCents = money.query(MonetaryQueries.convertMinorPart());//12023
        Long majorPart = money.query(MonetaryQueries.extractMajorPart());//120
        Long minorPart = money.query(MonetaryQueries.extractMinorPart());//23
    }
}
```

### MonetaryQuery vs MonetaryOperator 


Pero con ```MonetaryQuery``` es posible también retornar ```MonetaryAmount``` y asi tenemos el mismo resultado que  ```MonetaryOperator```, y cual es el punto de tener dos interfaces? El punto de tener las dos interfaces es por cuestión de nomenclatura y estandarización. ```MonetaryQuery``` tiene el objetivo seleccionar y buscar informaciones dentro de  ```MonetaryAmount```, ya ```MonetaryOperator``` tiene el objetivo de realizar operaciones con dinero.

```java
public class DifferenceMonetaryQueryMonetaryOperator {

    public  static void main(String[] args) {

        MonetaryQuery<MonetaryAmount> doubleQuery = m -> m.multiply(2);
        MonetaryOperator doubleOperator = m -> m.multiply(2);

    }
}
```
