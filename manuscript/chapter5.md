## Cotización


La internacionalización es un concepto aplicado en diversas áreas, básicamente, es un intercambio de cultura, economía y política en diversos lugares del mundo. Con la internacionalización es posible, por ejemplo, saber lo que sucede del otro lado del mundo de manera instantanea, conocer lugares y culturas sin salir de casa ademas de adquirir productos. Es muy común exportar e importar diversas cosas, productos, servicios, asignaturas, etc. Siguiendo la tendencia mundial los software también necesitan estar preparados para una arquitetura internacionalizada, osea, permitir la interacción del usuario en diversos lugares del mundo. Com dinero no es diferente, es necesario estar listo para esto.

Hablando de internacionalización y dinero es natural que necesite interactuar con diversas monedas, pero que pasa al realizar operaciones con monedas diferentes?


```java
public class MistakeExample {


    public static void main(String[] args) {
        CurrencyUnit dollar = Monetary.getCurrency("USD");
        CurrencyUnit real = Monetary.getCurrency("BRL");
        MonetaryAmount money = FastMoney.of(10, dollar);
        MonetaryAmount money2 = FastMoney.of(10, real);
        MonetaryAmount result = money.add(money2);//javax.money.MonetaryException: Currency mismatch: USD/BRL
    }
}
```


Utilizando money-api, este generará una excepción afirmando que existe um error al probar sumar dinero con monedas diferentes, en este caso real con dólar. Esta excepción fue lanzada de forma correcta ya que no necesariamente un dolar equivale a un real. Para realizar tal operación de sumatoria entre monedas diferentes es necesario primero realizar una cotización de la moneda. El tipo de cambio es la relación de las monedas entre dos paises que resulta en el precio de una de ellas con relación a otra. 

### Realizando cotización con ExchangeRateProvider

Con money-api es posible realizar la cotización de la moneda, el responsable por esa acción es la interface  ```ExchangeRateProvider```.

```java
public class ExchangeRateProviderExample {

    public static void main(String[] args) {
        CurrencyUnit dollar = Monetary.getCurrency("USD");
        CurrencyUnit real = Monetary.getCurrency("BRL");
        MonetaryAmount money = FastMoney.of(10, dollar);
        MonetaryAmount money2 = FastMoney.of(10, real);
        ExchangeRateProvider provider = MonetaryConversions.getExchangeRateProvider(ExchangeRateType.ECB);
        CurrencyConversion currencyConversion = provider.getCurrencyConversion(dollar);
        MonetaryAmount result = currencyConversion.apply(money2);//value on dollar
        MonetaryAmount monetaryAmount = money.add(result);//result on dollar
    }
}
```


Dentro de Moneta existen cuatro implementaciones de ```ExchangeRateProvider```:

* **ECB** implementación que recupera información reciente del Banco Central Europeo.
* **IMF** implementación que recupera las cotizaciones mas recientes del Fondo Internacional Monetario.
* **IMF_HIST** implementación que permite recuperar cotización de una fecha específica a partir del IMF.
* **ECB_HIST90** implementación que recupera los últimos noventa dias del Banco Central Europeo.
* **ECB_HIST** implementación que recupera las cotizaciones desde 1999 del Banco Central Europeo.

### Realizando la cotización a partir de una fecha especifica

En algunos momentos de la aplicación es importante saber no solo el valor de la cotización actual, sino a partir de una fecha especifica, por ejemplo, al alquilar un hotel normalmente el valor de la cotización es dado a partir de la confirmación de la reserva o en el caso de la tarjeta de crédito el valor de la cotización es definido solo en el cierre de la factura. Con Moneta es posible realizar tal búsqueda a partir de una fecha especifica para eso se utiliza la clase ```ConversionQuery``` con esta es posible realizar búsquedas de fechas diferentes o en un rango de fechas. La representación aceptada de fecha es la clase ```LocalDate```.


```java
public class ExchangeRateProviderExample2 {

    public static void main(String[] args) {

        CurrencyUnit dollar = Monetary.getCurrency("USD");
        CurrencyUnit real = Monetary.getCurrency("BRL");

        MonetaryAmount money = FastMoney.of(10, dollar);
        MonetaryAmount money2 = FastMoney.of(10, real);


        LocalDate localDate = Year.of(2009).atMonth(Month.JANUARY).atDay(9);

        ExchangeRateProvider provider = MonetaryConversions.getExchangeRateProvider(ExchangeRateType.IMF_HIST);
        ConversionQuery query = ConversionQueryBuilder.of().setTermCurrency(dollar).set(localDate).build();

        CurrencyConversion currencyConversion = provider.getCurrencyConversion(query);

        MonetaryAmount result = currencyConversion.apply(money2);
        MonetaryAmount monetaryAmount = money.add(result);
        System.out.println(monetaryAmount);
    }
}
```

Si la fecha especificada no sea encontrada será retornada una excepción, por ejemplo, no será posible recuperar la cotización del dia 9 de enero de 2011, ya que esa fecha fue en un domingo y la gran mayoria de los proveedores de cotización no trabajan en ese dia.

```java
public class ExchangeRateProviderExample3 {

    public static void main(String[] args) {

        CurrencyUnit dollar = Monetary.getCurrency("USD");
        CurrencyUnit real = Monetary.getCurrency("BRL");

        MonetaryAmount money = FastMoney.of(10, dollar);
        MonetaryAmount money2 = FastMoney.of(10, real);


        LocalDate localDate = Year.of(2011).atMonth(Month.JANUARY).atDay(9);

        ExchangeRateProvider provider = MonetaryConversions.getExchangeRateProvider(ExchangeRateType.IMF_HIST);
        ConversionQuery query = ConversionQueryBuilder.of().setTermCurrency(dollar).set(localDate).build();

        CurrencyConversion currencyConversion = provider.getCurrencyConversion(query);

        MonetaryAmount result = currencyConversion.apply(money2);//javax.money.MonetaryException: There is not exchange on day 2011-01-09 to rate to  rate on IFMRateProvider.


    }
}
```


Una posible solución para este problema es pasar un rango de fechas, asi la implementación va probar alguna de las fechas, si no encuentra ninguna de ellas lanzará una excepción, vale recalcar que la implementación buscará a partir de la orden de la que fue definida.

```java
public class ExchangeRateProviderExample4 {

    public static void main(String[] args) {

        CurrencyUnit dollar = Monetary.getCurrency("USD");
        CurrencyUnit real = Monetary.getCurrency("BRL");

        MonetaryAmount money = FastMoney.of(10, dollar);
        MonetaryAmount money2 = FastMoney.of(10, real);


        LocalDate localDate = Year.of(2011).atMonth(Month.JANUARY).atDay(9);
        LocalDate[] localDates = Stream.of(localDate, localDate.minusDays(1L), localDate.minusDays(2L),
                localDate.minusDays(3L)).sorted(Comparator.<LocalDate>naturalOrder().reversed()).toArray(LocalDate[]::new);
        ExchangeRateProvider provider = MonetaryConversions.getExchangeRateProvider(ExchangeRateType.IMF_HIST);
        ConversionQuery query = ConversionQueryBuilder.of().setTermCurrency(dollar).set(localDates).build();

        CurrencyConversion currencyConversion = provider.getCurrencyConversion(query);

        MonetaryAmount result = currencyConversion.apply(money2);
        MonetaryAmount sum = money.add(result);

    }
}
```

### Ignorando la cotización

En algunos momentos es necesario realizar un intercambio de moneda sin realizar la cotización, para eso existe el método **exchange** dentro de ```ConversionOperators```.

```java
public class ExchangeRateProviderExample5 {

    public static void main(String[] args) {

        CurrencyUnit dollar = Monetary.getCurrency("USD");
        CurrencyUnit real = Monetary.getCurrency("BRL");

        MonetaryAmount money = FastMoney.of(10, dollar);
        MonetaryAmount money2 = FastMoney.of(10, real);


        MonetaryOperator operator = ConversionOperators.exchange(dollar);
        MonetaryAmount result = money2.with(operator).add(money);//USD 20.00000 ignoring currency
    }
}
```